---
layout: post
title: Введение в Spring Data
---

Перевод [статьи](https://www.infoq.com/articles/spring-data-intro)


 Spring Data - это выскоуровневый проект целью которого является обьединить и легко использовать доступ к различными видам хранения, как реляционных баз данных, так и nosql
 
 ![]({{site.baseurl}}/https://res.infoq.com/articles/spring-data-intro/en/resources/spring_data_overview_small.jpg)
 
 C каждым видом хранения репозитории(так называемые DAO) типично предлагают CRUD операции над обьектами предметной области, поисковые методы, сортировку и пагинацию. Spring Data предоставляет общие интерфейсы  для этих аспектов(CrudRepository, PagingAndSortingRepository), в то время как персистентность содержит специфичные имплементации. 
 
 Template обьекты - являются довольно мощными и позволяют писать свои имплементации репозиториев. Но со репозиториями Spring Data возможно писать одни интерфейсы с поисковыми методами, сигнатура которых удовлетворяет некоторым конвенциям(которые могут различаться в зависимости от вида персистентности)
 
 В качестве примера: 
 {% highlight java %}
    public interface UserRepository extends MongoRepository<User, String> { 
        @Query("{ fullName: ?0 }")
        List<User> findByTheUsersFullName(String fullName);

        List<User> findByFullNameLike(String fullName, Sort sort);
}
...

@Autowired 
UserRepository repo;
{% endhighlight %} 


Рассмотрим подпроекты Spring Data для JPA, MongoDB, Neo4j. JPA - это часть JEE-стека и опредлеляет единый API для доступа к реляционным базам данных и предоставляя ORM. Все эти проекты поддерживают следующие аспекты:

- Templating
- Object/Datastore mapping
- Repository support
    
Дргуие проекты предоствляют только templtes, так как соответствующие им базы данных хранят неструктуированные данные которые не могут быть отображены через запросы или преобразованы в обьекты.

Templates

Главная цель templates (и всех других видов Spring Template) это аллокация ресурсов и преобразование исключений.

В случае MongoDB проект на нижнем уровне зависит от Mongo DB Java Driver. В общем случае такой низкий уровень API будет иметь свою стратегию обработки исключений. Spring way обработки исключений - использовать непроверяемые исключения которые разработчик может поймать, но не должен. Чтобы покрыть этот промежуток имплементация template ловит нижний уровень исключений и перебрасывает соответствующее непроверяемое Spring-исключение, которое является наследником Spring's DataAccessException

Template чаще всего предоствляет специфичесике операции как сохранение, обновление, удаление или map/reduce запросы. Но эти методы работают только на для соответствующей базы данных которая лежит на нижнем уровне.

Spring Data JPA не предоставлет template так как основывается на JDBC API, для которого template существует. JPA's EntityManager - является эквивалентом темплейта. 

Mapping Objects

JPA ввел стандарт для ORM.(так называемое отображение связей обьектов на реляционные таблицы). Hibernate - возможно наиболее распостранненый O/R Mapper который имплментирует JPA спецификацию. 


С Spring Data эта возможноcть расширилась до NoSql баз данных с обьектной структурой данных. Но это струкрутра данных может сильно различаться дргу от друга и поэтому это было бы сложно сделать общий API для маппинга. 
Каждое хранилище поставляется с собственным набором аннтоаций чтобы предоставить необходимую мета-информацию для маппинга. Для примера:

{% highlight java %}

JPA

@Entity
@Table(name="TUSR")
public class User {

  @Id
  private String id;

  @Column(name="fn")
  private String name;

  private Date lastLogin;

...
}

MongoDB	

@Document(
collection="usr")
public class User {

  @Id
  private String id;

  @Field("fn")
  private String name;

  private Date lastLogin;

 ...
}

Neo4j	
    
    

@NodeEntity
public class User {

  @GraphId
  Long id;


  private String name;

  private Date lastLogin;

...
}
{% endhighlight %}
 

 Spring Data переиспользует JPA-аннтации в первом случае и не вводит дргуие. Отображение завист от конкретной имплементации jpa.Для  MongoDB and Neo4j требуется похожий набор аннотаций. Значения этих атрибутов будут сохранены под именем переменных. Или можно использовать аннтоацию @Field( для mongodb)
 
Связи между обьектамии также поддерживаются. Роли пользователей могут храниться в этом виде:
{% highlight java %}
JPA

@OneToMany
private List<Role> roles;

MongoDB
	

private List<Role> roles;

Neo4j

@RelatedTo(
type = "has",
direction = 
Direction.OUTGOING)
private List<Role> roles;
{% endhighlight %}
  
С JPA можно использовать @OneToMany отношения, n-сторон обычно хранятся в другой таблицу и обычно запрашиваются с помощбю join. MongoDB не поддерживает join между коллекциями, по умолчанию ссылочные обьекты  хранятся  в том же документе. Ты можешь иметь ссылки на другие документы которые привоядт к соединениям на стороне клиента. В Neo4J отношения называются ребрами которые являются основными типами данных. 

Подводя итоги: MongoDB и Neo4j используют маппинг обьектов который похож на реляционный маппинг JPA, но это не одно и то же из-за разных структур данных. Концепт который стоит за сопоставлением впрочем один и тот же: сопоставлять Java-обьекты к структурах данных в используемом хранилище.

Поддержка Reppository

Если вам когда-нибудь приходилось сохранять данные в вашем бизнес-приложении вы возможно писало что-то вроде DAO. Обычно вы имплементировали CRUD операции для отдельных записей и кучу поисковых методов для каждой сохраняемой сущности. Поисковые методы принмали параметры, которые вы клали в запрос, перед тем как исполнить его. С появлением JPA методы поиска по крайней мере достуны через интерфйес EntityManager. Тем не менее писать собственные методы поиска все еще утомительно: создать именованный запрос, вставить каждый параметр, выполнить запрос. Например:
{% highlight java %}
@Entity
@NamedQuery( name="myQuery", query = "SELECT u FROM User u where u.name = :name" )
public class User { 
...
} 

@Repository 
public class ClassicUserRepository { 

   @PersistenceContext EntityManager em; 

   public List<User> findByName(String Name) { 
      TypedQuery<User> q = getEntityManger().createNamedQuery("myQuery", User.class); 

      q.setParameter("name", fullName);

      return q.getResultList();
   } 
   ...
{% endhighlight %}

Этот пример может быть немного сокращен если использовать интерфейс TypedQuery ...

{% highlight java %}
@Repository
public class ClassicUserRepository { 

   @PersistenceContext EntityManager em; 

   public List<User> findByName(String name) {
      return getEntityManger().createNamedQuery("myQuery", User.class)
         .setParameter("name", fullName)
         .getResultList(); 
   } 
   ...
{% endhighlight %}
  
...но тем не менее вы имплементируете метод который вызывает сеттеры и выполняет запрос за запросом
С Spring Data JPA похожий запрос сводится к следующему куску кода. 

{% highlight java %}

package repositories; 

public interface UserRepository extends JpaRepository<User, String> {

   List<User> findByName(String name); 
}
  
{% endhighlight %}

При использовании Spring Data JPA, JPQL запросы не нужно декларирвать как @NamedQuery в файле класса с соответствующей JPA сущностью. Вместо этого запрос это аннтоация к методу реозитория(!).

{% highlight java %}

@Transactional(timeout = 2, propagation = Propagation.REQUIRED)
@Query("SELECT u FROM User u WHERE u.name = 'User 3'")
List<User> findByGivenQuery();

{% endhighlight %}

Все вышеописанное также справедливо и для MongoDB и Spring Data Neo4j.Следующий пример совершает запрос к базе данных Neo4j используя язык запросов Сipher
{% highlight java %}

public interface UserRepository extends GraphRepository<User> {

  User findByLogin(String login); 

  @Query("START root=node:User(login = 'root') MATCH root-[:knows]->friends RETURN friends")
  List<User> findFriendsOfRoot(); 
}
{% endhighlight %}
	
Конечно соглашения именования методов поиска отличаются от базе к базе. Для примера MongoDB поддерживает   геопространственные запросы. Их вы можете писать так:

{% highlight java %}

public interface LocationRepository extends MongoRepository<Location, String> {

        List<Location> findByPositionWithin(Circle c);

        List<Location> findByPositionWithin(Box b);

        List<Location> findByPositionNear(Point p, Distance d);
}
{% endhighlight %}

 Также существует обобщенная поддержка пагинации и сортировки, путем предоставления специалных параметров к поисковым методам хранилищ.
 
Главные преимущества поддержки репозиториев следующие:
- Разработчик пишет меньше бойлерплэйта 
- Запросы могут быть определены рядом с методами поиска и их документацией
- В качестве бонуса JPQL запросы компилируется как только будет собран контекст Spring, а не в первый раз когда вы используете запрос, что позволяет быстрее обнаруживать синтаксические ошибки

#### Итоги

Эта статья была введением во множество сложных технологий и попыталась выявить различия и схоие моменты. 
Возвращаясь к вопросу который был задан в заголовке(One api to tule them all?): нет, в Spring Data нет единого интерфейса ко всем хранилищам. Разница между ними слишком фундаментальна. Но проект Spring Data действительно предоставляет общую модель программирования для доступа к данным. 

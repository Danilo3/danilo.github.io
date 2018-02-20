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


С Spring Data эта возможно асширилась до NoSql баз данных с обьектной структурой данных. Но это струкрутра данных может сильно различаться дргу от друга и поэтому это было бы сложно сделать общий API для маппинга. 
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

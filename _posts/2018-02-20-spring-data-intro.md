---
published: true
layout: post
---
## Введение в Spring Data


 Spring Data - это выскоуровневый проект целью которого является обьединить и легко использовать доступ к различными видам хранения, как реляционных баз данных, так и nosql
 
 ![]({{site.baseurl}}/https://res.infoq.com/articles/spring-data-intro/en/resources/spring_data_overview_small.jpg)
 
 C каждым видом хранения репозитории(так называемые DAO) типично предлагают CRUD операции над обьектами предметной области, поисковые методы, сортировку и пагинацию. Spring Data предоставляет общие интерфейсы  для этих аспектов(CrudRepository, PagingAndSortingRepository), в то время как персистентность содержит специфичные имплементации. 
 
 Template обьекты - являются довольно мощными и позволяют писать свои имплементации репозиториев. Но со репозиториями Spring Data возможно писать одни интерфейсы с поисковыми методами, сигнатура которых удовлетворяет некоторым конвенциям(которые могут различаться в зависимости от вида персистентности)
 
 В качестве примера: 
 {% highlight java}
    
    
    public interface UserRepository extends MongoRepository<User, String> { 
        @Query("{ fullName: ?0 }")
        List<User> findByTheUsersFullName(String fullName);

        List<User> findByFullNameLike(String fullName, Sort sort);
}
...

@Autowired 
UserRepository repo;

{% endhighlight} 

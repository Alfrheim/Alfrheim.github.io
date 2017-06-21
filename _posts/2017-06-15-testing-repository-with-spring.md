---
layout: post
title:  "Testing "
date:   2017-06-15 02:16:01 +0200
categories: java spring
---
Recently I wanted to test the queries I was doing in the Database. The project was using spring boot and jdbcTemplate in a H2 database.

Since it is an Integration Test I need to attack a real database or in this case an in memory DB.

So, first things first. Create a maven project and add spring jdbc and h2 libraries. 

{% highlight maven %}
<dependencies>
  ...
  <dependency>
    <groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
	<version>4.3.4.RELEASE</version>
  </dependency>
  <dependency>
    <groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
	<version>1.3.156</version>
	<scope>test</scope>
  </dependency>
  ...
</dependencies>
{% endhighlight %}

Then we create a script that will create the table in the database.

{% highlight sql %}
CREATE TABLE USERS (
  ID INT AUTO_INCREMENT PRIMARY KEY,
  NAME VARCHAR(255)
);
{% endhighlight %}

Next, we create a Test that checks that when we add an username we can retrieve it after. 

{% highlight java %}
...

public class UserRepositoryShould {

    private UserRepository userRepository;

    @Before
    public void setUp() throws Exception {
        EmbeddedDatabase db = new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .generateUniqueName(true)
                .addScript("db/sql/create-db.sql")
                .build();

        this.userRepository = new UserRepository(new JdbcTemplate(db));
    }

    @Test
    public void increase_the_number_of_users_when_user_is_added() throws Exception {
        userRepository.createUser("Largo Winch");
        userRepository.createUser("Jhon Wick");

        List<String> users = userRepository.getAllUsers();

        assertThat(users.size())
                .isEqualTo(2);
    }
}

{% endhighlight %}

So, what we do in the setup?

We specify the type of database we are going to use and execute the sql script we just created.
And what about the *.generateUniqueName(true)*? you need that if you want to create a new database each time a test is run. By default the builder creates a bean named *test*. That means that if we had a second test, that one will run the *@Before* and instead creating a whole new database, will see that *test* exists and will reuse that one. 
No need to say that it will jump an exception saying that we already have the existing table and can't run the scpript. That can happen even if we try that in different Test files.

Now we only need to write the implementation of the methods and check that everything works fine.

{% highlight java %}
...
public class UserRepository {
    private final JdbcTemplate jdbcTemplate;

    public UserRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public List<String> getAllUsers() {
        return jdbcTemplate.queryForList("SELECT NAME FROM USERS", String.class);
    }

    public void createUser(String userName) {
        jdbcTemplate.update("INSERT INTO USERS(NAME) VALUES(?)", userName);
    }
...
{% endhighlight %}

You can see the whole code in https://github.com/Alfrheim/IntegrationTestSpringJdbc .

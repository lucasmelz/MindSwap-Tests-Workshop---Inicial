# MindSwap Software Testing Workshop

## Tutorial
1. Pull Postgres Docker Image:<br>
`docker pull postgres`


2. Run the docker container:<br>

`docker run --name myPostgresDb -p 5455:5432 -e POSTGRES_USER=user -e POSTGRES_PASSWORD=password -e POSTGRES_DB=postgresDB -d postgres`

3. Run a bash on the container:

`docker exec -it myPostgresDb bash`


4. Execute PostgreSQL with the role "user"<br>
`psql postgresDB -U user`


5. Create database: <br>
`create database test; `<br>
Optional: You can list all databases with `\l` and connect to the test database with the command `\c test`. You can list all the tables with the command `\dt`.


6. Confirm that your application.properties has the following lines:

<pre><code>spring.datasource.url=jdbc:postgresql://localhost:5455/test
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.username=user
spring.datasource.password=password
</code></pre>

7. Create test "StudentRepositoryTest" for Student Repository and add this method to it. You can create the test class by going to the StudentRepository class and pressing COMMAND + SHIFT + T and selecting the appropriate options.

    ```
    @Test
    void itShouldCheckIfStudentExistsEmail() {
        
    }
8. Add StudentRepository undertest to the beggining of the test class
   ```
    @Autowired
    private StudentRepository underTest
9. Add 
    ```
    @Test
    void itShouldCheckIfStudentEmailExists() {
        //given
        String email = "jamila@gmail.com";
        Student student = new Student(
            "Jamila",
            email,
            Gender.FEMALE
        );
        //when
        underTest.save(student);

        //then
        boolean expected = underTest.selectExistsEmail(email);
        
        assertThat(expected).isTrue();
    }
10. Add this dependency to pom and Reload Maven
    ```
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>

11. Copy application.properties from src/**main**/resources/ to src/**test**/resources/ (Create the resources directory if it doesn't exist).
12. Copy this into the file src/test/resources/application.properties
```
spring.datasource.url=jdbc:h2://mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true
```
13. Run the test again. Watch it fail
14. Add @DataJpaTest to the header above ```class StudentRepositoryTest```    
15. Run again. Check that it works.
16. Create this test. And run it afterwards
   ```
    @Test
    void itShouldCheckIfStudentEmailDoesNotExist() {
        //given
        String email = "jamila@gmail.com";
        
        //when
        boolean expected = underTest.selectExistsEmail(email);
        
        //then
        assertThat(expected).isFalse();
    }
```
17. Create this tearDown
    ```
        @AfterEach
        void tearDown() {
            underTest.deleteAll();
        }

18. Say that we are only testing these methods, because these are methods that were created. There are loads of methods that were provided by Spring and already tested. We only have to test the methods in our interface.

19. First test Service the wrong way. Create test "StudentServiceTest" for Student Service

20. Generate relevant testing methods. You can do this automatically by going to IntelliJ, right click anywhere on StudentService and clicking Generate tests (or using SHIFT + COMMAND + T).

21. Add ```private StudentService undertest;``` <br>

22. We need a repository to test our service class. Since we already tested our repository and we know it works, instead of auto wiring a repository using an in memory database, we want to use a mock repository. Add ```@Mock private StudentRepository studentRepository;```

     
24. Add a *setUp* method in the beginning of the class that gives us a fresh instance of a StudentService object for each test.

    ```
    @Test
    void setUp() {
        underTest = new StudentService(studentRepository)
    }
    ```

25. Add @ExtendWith(MockitoExtension.class) annotation to the StudentServiceTest. This annotation basically opens up our mock repository before each test and closes it after each test.
    ```
    @ExtendWith(MockitoExtension.class)
    class StudentServiceTest {
    ```

26. Add/modify method canGetAllStudents for the real test

    ```
    @Test
    void canGetAllStudents() {
        //when
        underTest.getAllStudents();
        //then
        verify(studentRepository).findAll();
    }
    ```

27. Add method canAddStudent

    ```
    @Test
    void canAddStudent() {
        //given
        Student student = new Student(
            "Jamila",
            "jamila@gmail.com",
            Gender.FEMALE
        );
        //when
        undertest.addStudent(student)
        // then
        ArgumentCaptor<Student> studentArgumentCaptor =
                ArgumentCaptor.forClass(Student.class);

        verify(studentRepository)
                .save(studentArgumentCaptor.capture());

        Student capturedStudent = studentArgumentCaptor.getValue();

        assertThat(capturedStudent).isEqualTo(student);
    }
    ```


28. Show code coverage. Go to run and select "Generate Test Coverage". Go to StudentService and check the left side for green and red blocks.


29. Go fix that Red block of coverage by testing it.

```
    @Test
    void willThrowEmailIfTaken() {
        //given
        Student student = new Student(
            "Jamila",
            "jamila@gmail.com",
            Gender.FEMALE
        );

        given(studentRepository.selectExistsEmail(student.getEmail()))
        .willReturn(true);
        //when
        //then

        assertThatThrownBy(() -> undertest.addStudent(student))
        .isInstanceOf(BadRequestException.class)
        .hasMessageContaining("Student with id " + studentId + " does not exists");

        verify(studentRepository, never()).save(any());
        
    }
```

30. Let's now write our first integration tests. An integration test is made to test how different the different components of the system interact with each other. Let's first create a package called *integration* inside *test.java.com.example.demo*.

31. Copy the *application.properties* from the main.java.* directory and paste it inside the *resources* folder inside the test folder. Rename it as *application-it.properties*.

32. Create a class inside the integration package we created at step 30. Call it *StudentIt*. This class should have the following annotations and instance variables:
``` 
    @SpringBootTest
    @TestPropertySource(
        locations = "classpath:application-it.properties"
    )
    @AutoConfigureMockMvc
    public class StudentIT {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private StudentRepository studentRepository;

    private final Faker faker = new Faker();
    ...
```

33. Let's test if our application is able to delete students. Write the following test:
```
    @Test
    void canDeleteStudent() throws Exception {
    // given
    String name = String.format(
    "%s %s",
    faker.name().firstName(),
    faker.name().lastName()
    );

    String email = String.format("%s@amigoscode.edu",
                StringUtils.trimAllWhitespace(name.trim().toLowerCase()));

    Student student = new Student(
                name,
                email,
                Gender.FEMALE
    );

    mockMvc.perform(post("/api/v1/students")
           .contentType(MediaType.APPLICATION_JSON)
           .content(objectMapper.writeValueAsString(student)))
           .andExpect(status().isOk());


    MvcResult getStudentResult = mockMvc.perform(get("/api/v1/students/getByEmail?email=" + email)
              .contentType(MediaType.APPLICATION_JSON))
              .andExpect(status().isOk())
              .andReturn();

    String contentAsString = getStudentResult
                .getResponse()
                .getContentAsString();

    Student studentReceived = objectMapper.readValue(
                contentAsString,
                new TypeReference<>() {
                }
    );

    if (studentReceived==null){
        throw new IllegalStateException("student with email: " + email + " not found");
    }
    long id = studentReceived.getId();

    // when
    ResultActions resultActions = mockMvc
                .perform(delete("/api/v1/students/" + id));

    // then
    resultActions.andExpect(status().isOk());
    boolean exists = studentRepository.existsById(id);
    assertThat(exists).isFalse();
    }
  }
```
    

## Exercises 

1. We already tested if our application is capable to reject the registration of a student if it has the same email as another already registered. Now we want to make sure that it only registers students with a valid email. This exercise consists in modifying the method *addStudent* of the StudentService class, by making it to throw an exception if a student tries to register with an invalid email. Then we want to create the corresponding test to cover this requirement. We are going to use the Apache Commons Validator package to validate emails. So the first step is to add this dependency to our pom.xml file:
```
		<dependency>
			<groupId>commons-validator</groupId>
			<artifactId>commons-validator</artifactId>
			<version>1.6</version>
		</dependency>
```
Then reload the project with Maven for it to recognize Apache Commons Validator. Let's modify the method addStudent of the StudentService class now. First we need an object of the type EmailValidator. You can initialize it inside the method like this:<br>
`EmailValidator emailValidator = EmailValidator.getInstance();`<br>
Now you can use the method *isValid(String email)* provided by this object to validate emails. Make sure you throw a *BadRequestException* if the email is not valid, just like it does when the email already is taken. After you do that, you can use Intellij Idea test coverage tool to note that this code we just wrote is not being covered by our tests. So let's fix that.

Let's create the corresponding test. Go to the StudentServiceTest class and create a method *willThrowWhenEmailIsInvalid()* and implement the test in a similar fashion as the method *willThrowWhenEmailIsTaken()*.

2. Similarly to the integration test *canDeleteStudent* created at the step 33 of the tutorial, create an integration test called *canRegisterNewStudent*, whose name is self-explanatory...

# Acknowledgments
# Unit Testing, a First Example 
```java
public class CustomerTest {
  Customer john, steve, pat, david;
  String johnName = "John",
    steveName = "Steve",
    patName = "Pat",
    davidName = "David";
  Customer[] customers;

  @Before
  public void setup() {
    david = ObjectMother
      .customerWithNoRentals(
        davidName);
    john = ObjectMother
      .customerWithOneNewRelease(
        johnName);
    pat = ObjectMother
      .customerWithOneOfEachRentalType(
        patName);
    steve = ObjectMother
      .customerWithOneNewReleaseAndOneRegular(
        steveName);
    customers =
      new Customer[]
      { david, john, steve, pat};
  }
```

```java
  @Test
  public void getName() {
    assertEquals(
      davidName, david.getName());
    assertEquals(
      johnName, john.getName());
    assertEquals(
      steveName, steve.getName());
    assertEquals(
      patName, pat.getName());
  }

  @Test
  public void statement() {
    for (int i=0; i<customers.length; i++) {
      assertEquals(
        expStatement(
          "Rental record for %s\n" +
          "%sAmount owed is %s\n"  +
          "You earned %s frequent " +
          "renter points",
          customers[i],
          rentalInfo(
            "\t", "",
            customers[i].getRentals())),
        customers[i].statement());
    }
  }
```

```java
  @Test
  public void htmlStatement() {
    for (int i=0; i<customers.length; i++) {
      assertEquals(
        expStatement(
          "<h1>Rental record for " +
          "<em>%s</em></h1>\n%s" +
          "<p>Amount owed is <em>%s</em>" +
          "</p>\n<p>You earned <em>%s" +
          " frequent renter points</em></p>",
          customers[i],
          rentalInfo(
            "<p>", "</p>",
            customers[i].getRentals())),
        customers[i].htmlStatement());
    }
  }

  @Test
  (expected=IllegalArgumentException.class)
  public void invalidTitle() {
    ObjectMother
      .customerWithNoRentals("Bob")
      .addRental(
        new Rental(
          new Movie("Crazy, Stupid, Love.",
                    Movie.Type.UNKNOWN),
          4));
  }
```

```java
  public static String rentalInfo(
    String startsWith,
    String endsWith,
    List<Rental> rentals) {
    String result = "";
    for (Rental rental : rentals)
      result += String.format(
        "%s%s\t%s%s\n",
        startsWith,
        rental.getMovie().getTitle(),
        rental.getCharge(),
        endsWith);
    return result;
  }

  public static String expStatement(
    String formatStr,
    Customer customer,
    String rentalInfo) {
    return String.format(
      formatStr,
      customer.getName(),
      rentalInfo,
      customer.getTotalCharge(),
      customer.getTotalPoints());
  }
}
```
```java
public class ObjectMother {
  public static Customer
  customerWithOneOfEachRentalType(
    String name) {
    Customer result =
      customerWithOneNewReleaseAndOneRegular(
        name);
    result.addRental(
      new Rental(
        new Movie("Lion King", CHILDREN), 3));
    return result;
  }

  public static Customer
  customerWithOneNewReleaseAndOneRegular(
    String n) {
    Customer result =
      customerWithOneNewRelease(n);
    result.addRental(
      new Rental(
        new Movie("Scarface", REGULAR), 3));
    return result;
  }
```

```java
  public static Customer
  customerWithOneNewRelease(
    String name) {
    Customer result =
      customerWithNoRentals(name);
    result.addRental(
      new Rental(
        new Movie(
          "Godfather 4", NEW_RELEASE), 3));
    return result;
  }

  public static Customer
  customerWithNoRentals(String name) {
    return new Customer(name);
  }
}
```
##  Thoughts on our Tests
## The Domain Code
```java
public class Customer {

  private String name;
  private List<Rental> rentals =
    new ArrayList<Rental>();

  public Customer(String name) {
    this.name = name;
  }

  public String getName() {
    return name;
  }

  public List<Rental> getRentals() {
    return rentals;
  }

  public void addRental(Rental rental) {
    rentals.add(rental);
  }
```

```java
  public String statement() {
    String result =
      "Rental record for " + getName() + "\n";
    for (Rental rental : rentals)
      result +=
        "\t" + rental.getLineItem() + "\n";
    result +=
      "Amount owed is " + getTotalCharge() +
      "\n" + "You earned " +
      getTotalPoints() +
      " frequent renter points";
    return result;
  }

  public String htmlStatement() {
    String result =
      "<h1>Rental record for <em>" +
      getName() + "</em></h1>\n";
    for (Rental rental : rentals)
      result += "<p>" + rental.getLineItem() +
        "</p>\n";
    result +=
      "<p>Amount owed is <em>" +
      getTotalCharge() + "</em></p>\n" +
      "<p>You earned <em>" +
      getTotalPoints() +
      " frequent renter points</em></p>";
    return result;

  }
```

```java
  public double getTotalCharge() {
    double total = 0;
    for (Rental rental : rentals)
      total += rental.getCharge();
    return total;
  }

  public int getTotalPoints() {
    int total = 0;
    for (Rental rental : rentals)
      total += rental.getPoints();
    return total;
  }
}
```
```java
public class Rental {

  Movie movie;
  private int daysRented;

  public Rental(Movie movie, int daysRented) {
    this.movie = movie;
    this.daysRented = daysRented;
  }

  public Movie getMovie() {
    return movie;
  }

  public int getDaysRented() {
    return daysRented;
  }

  public double getCharge() {
    return movie.getCharge(daysRented);
  }

  public int getPoints() {
    return movie.getPoints(daysRented);
  }

  public String getLineItem() {
    return
      movie.getTitle() + " " + getCharge();
  }
}
```
```java
public class Movie {

  public enum Type {
    REGULAR, NEW_RELEASE, CHILDREN, UNKNOWN;
  }

  private String title;
  Price price;

  public Movie(
    String title, Movie.Type priceCode) {
    this.title = title;
    setPriceCode(priceCode);
  }

  public String getTitle() {
    return title;
  }
```

```java
  private void setPriceCode(
    Movie.Type priceCode) {
    switch (priceCode) {
    case CHILDREN:
      price = new ChildrensPrice();
      break;
    case NEW_RELEASE:
      price = new NewReleasePrice();
      break;
    case REGULAR:
      price = new RegularPrice();
      break;
    default:
      throw new IllegalArgumentException(
        "invalid price code");
    }
  }

  public double getCharge(int daysRented) {
    return price.getCharge(daysRented);
  }

  public int getPoints(int daysRented) {
    return price.getPoints(daysRented);
  }
}
```
```java
public abstract class Price {
  abstract double getCharge(int daysRented);

  int getPoints(int daysRented) {
    return 1;
  }
}
```
```java
public class ChildrensPrice extends Price {
  @Override
  double getCharge(int daysRented) {
    double amount = 1.5;
    if (daysRented > 3)
      amount += (daysRented - 3) * 1.5;
    return amount;
  }
}
```
```java
public class RegularPrice extends Price {
  @Override
  public double getCharge(int daysRented) {
    double amount = 2;
    if (daysRented > 2)
      amount += (daysRented - 2) * 1.5;
    return amount;
  }
}
```
```java
public class NewReleasePrice extends Price {
  @Override
  public double getCharge(int daysRented) {
    return daysRented * 3;
  }

  @Override
  int getPoints(int daysRented) {
    if (daysRented > 1)
      return 2;
    return 1;
  }
}
```
## Moving Towards Readability
## Replace Loop with Individual Tests
```java
public class CustomerTest {
  Customer john, steve, pat, david;
  String johnName = "John",
    steveName = "Steve",
    patName = "Pat",
    davidName = "David";
  Customer[] customers;

  @Before
  public void setup() {
    david = ObjectMother
      .customerWithNoRentals(davidName);
    john = ObjectMother
      .customerWithOneNewRelease(johnName);
    pat = ObjectMother
      .customerWithOneOfEachRentalType(
        patName);
    steve = ObjectMother
      .customerWithOneNewReleaseAndOneRegular(
        steveName);
    customers = new Customer[] {
      david, john, steve, pat };
  }
```

```java
  
  @Test
  public void davidStatement() {
    assertEquals(
      expStatement(
        "Rental record for %s\n%sAmount " +
        "owed is %s\nYou earned %s " +
        "frequent renter points",
        david,
        rentalInfo(
          "\t", "", david.getRentals())),
      david.statement());
  }

  @Test
  public void johnStatement() {
    assertEquals(
      expStatement(
        "Rental record for %s\n%sAmount " +
        "owed is %s\nYou earned %s " +
        "frequent renter points",
        john,
        rentalInfo(
          "\t", "", john.getRentals())),
      john.statement());
  }
```

```java
  @Test
  public void patStatement() {
    assertEquals(
      expStatement(
        "Rental record for %s\n%sAmount " +
        "owed is %s\nYou earned %s " +
        "frequent renter points",
        pat,
        rentalInfo(
          "\t", "", pat.getRentals())),
      pat.statement());
  }

  @Test
  public void steveStatement() {
    assertEquals(
      expStatement(
        "Rental record for %s\n%s" +
        "Amount owed is %s\nYou earned %s " +
        "frequent renter points",
        steve,
        rentalInfo(
          "\t", "", steve.getRentals())),
      steve.statement());
  }
```

```java
  

  public static String rentalInfo(
    String startsWith,
    String endsWith,
    List<Rental> rentals) {
    String result = "";
    for (Rental rental : rentals)
      result += String.format(
        "%s%s\t%s%s\n",
        startsWith,
        rental.getMovie().getTitle(),
        rental.getCharge(),
        endsWith);
    return result;
  }

  public static String expStatement(
    String formatStr,
    Customer customer,
    String rentalInfo) {
    return String.format(
      formatStr,
      customer.getName(),
      rentalInfo,
      customer.getTotalCharge(),
      customer.getTotalPoints());
  }
}
```
## Expect Literals
```java
public class CustomerTest {
  Customer john, steve, pat, david;
  String johnName = "John",
    steveName = "Steve",
    patName = "Pat",
    davidName = "David";
  Customer[] customers;

  @Before
  public void setup() {
    david = ObjectMother
      .customerWithNoRentals(davidName);
    john = ObjectMother
      .customerWithOneNewRelease(johnName);
    pat = ObjectMother
      .customerWithOneOfEachRentalType(
        patName);
    steve = ObjectMother
      .customerWithOneNewReleaseAndOneRegular(
        steveName);
    customers = new Customer[] {
      david, john, steve, pat };
  }

  @Test
  public void davidStatement() {
    assertEquals(
      
      "Rental record for David\nAmount " +
      "owed is 0.0\n" +
      "You earned 0 frequent renter points",
      
      david.statement());
  }
```

```java
  @Test
  public void johnStatement() {
    assertEquals(
      
      "Rental record for John\n\t" +
      "Godfather 4\t9.0\n" +
      "Amount owed is 9.0\n" +
      "You earned 2 frequent renter points",
      
      john.statement());
  }

  @Test
  public void patStatement() {
    assertEquals(
      
      "Rental record for Pat\n\t" +
      "Godfather 4\t9.0\n" +
      "\tScarface\t3.5\n" +
      "\tLion King\t1.5\n" +
      "Amount owed is 14.0\n" +
      "You earned 4 frequent renter points",
      
      pat.statement());
  }
```

```java
  @Test
  public void steveStatement() {
    assertEquals(
      
      "Rental record for Steve\n\t" +
      "Godfather 4\t9.0\n" +
      "\tScarface\t3.5\n" +
      "Amount owed is 12.5\n" +
      "You earned 3 frequent renter points",
      
      steve.statement());
  }
}
```
## Inline Setup
```java
public class CustomerTest {
  @Test
  public void noRentalsStatement() {
    assertEquals(
      "Rental record for David\nAmount " +
      "owed is 0.0\n" +
      "You earned 0 frequent renter points",
      
      ObjectMother
      .customerWithNoRentals(
        "David").statement());
    
  }

  @Test
  public void oneNewReleaseStatement() {
    assertEquals(
      "Rental record for John\n\t" +
      "Godfather 4 9.0\n" +
      "Amount owed is 9.0\n" +
      "You earned 2 frequent renter points",
      
      ObjectMother
      .customerWithOneNewRelease(
        "John").statement());
    
  }
```

```java
  @Test
  public void allRentalTypesStatement() {
    assertEquals(
      "Rental record for Pat\n\t" +
      "Godfather 4 9.0\n" +
      "\tScarface 3.5\n\tLion King 1.5\n" +
      "Amount owed is 14.0\n" +
      "You earned 4 frequent renter points",
      
      ObjectMother
      .customerWithOneOfEachRentalType(
        "Pat").statement());
    
  }

  @Test
  public void
  newReleaseAndRegularStatement() {
    assertEquals(
      "Rental record for Steve\n\t" +
      "Godfather 4 9.0\n" +
      "\tScarface 3.5\n" +
      "Amount owed is 12.5\n" +
      "You earned 3 frequent renter points",
      
      ObjectMother
      .customerWithOneNewReleaseAndOneRegular(
        "Steve").statement());
    
  }
}
```
## Replace ObjectMother with DataBuilder
```java
public class a {
  public static CustomerBuilder customer =
    new CustomerBuilder();
  public static RentalBuilder rental =
    new RentalBuilder();
  public static MovieBuilder movie =
    new MovieBuilder();

  public static class CustomerBuilder {
    Rental[] rentals;
    String name;

    CustomerBuilder() {
      this("Jim", new Rental[0]);
    }

    CustomerBuilder(
      String name, Rental[] rentals) {
      this.name = name;
      this.rentals = rentals;
    }

    public CustomerBuilder w(
      RentalBuilder... builders) {
      Rental[] rentals =
        new Rental[builders.length];
      for (int i=0; i<builders.length; i++) {
        rentals[i] = builders[i].build();
      }
      return
        new CustomerBuilder(name, rentals);
    }

    public CustomerBuilder w(String name) {
      return
        new CustomerBuilder(name, rentals);
    }

    public Customer build() {
      Customer result = new Customer(name);
      for (Rental rental : rentals) {
        result.addRental(rental);
      }
      return result;
    }
  }
```

```java
  public static class RentalBuilder {
    final Movie movie;
    final int days;

    RentalBuilder() {
      this(new MovieBuilder().build(), 3);
    }

    RentalBuilder(Movie movie, int days) {
      this.movie = movie;
      this.days = days;
    }

    public RentalBuilder w(
      MovieBuilder movie) {
      return
        new RentalBuilder(
          movie.build(), days);
    }

    public Rental build() {
      return new Rental(movie, days);
    }
  }
```

```java
  public static class MovieBuilder {
    final String name;
    final Movie.Type type;

    MovieBuilder() {
      this("Godfather 4",
           Movie.Type.NEW_RELEASE);
    }

    MovieBuilder(
      String name, Movie.Type type) {
      this.name = name;
      this.type = type;
    }

    public MovieBuilder w(Movie.Type type) {
      return new MovieBuilder(name, type);
    }

    public MovieBuilder w(String name) {
      return new MovieBuilder(name, type);
    }

    public Movie build() {
      return new Movie(name, type);
    }
  }
}
```
```java
public class CustomerTest {
  @Test
  public void noRentalsStatement() {
    assertEquals(
      "Rental record for David\nAmount " +
      "owed is 0.0\nYou earned 0 frequent " +
      "renter points",
      
      a.customer.w(
        "David").build().statement());
    
  }

  @Test
  public void oneNewReleaseStatement() {
    assertEquals(
      "Rental record for John\n\t" +
      "Godfather 4 9.0\nAmount owed is " +
      "9.0\nYou earned 2 frequent renter " +
      "points",
      
      a.customer.w("John").w(
        a.rental.w(
          a.movie.w(NEW_RELEASE))).build()
      
      .statement());
  }
```

```java
  @Test
  public void allRentalTypesStatement() {
    assertEquals(
      "Rental record for Pat\n\t" +
      "Godfather 4 9.0\n\tScarface 3.5\n" +
      "\tLion King 1.5\nAmount owed is " +
      "14.0\nYou earned 4 frequent renter " +
      "points",
      
      a.customer.w("Pat").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(a.movie.w("Scarface").w(
                     REGULAR)),
        a.rental.w(a.movie.w("Lion King").w(
                     CHILDREN))).build()
      
      .statement());
  }
```

```java
  @Test
  public void
  newReleaseAndRegularStatement() {
    assertEquals(
      "Rental record for Steve\n\t" +
      "Godfather 4 9.0\n\tScarface 3.5\n" +
      "Amount owed is 12.5\nYou earned 3 " +
      "frequent renter points",
      
      a.customer.w("Steve").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(
          a.movie.w(
            "Scarface").w(REGULAR))).build()
      
      .statement());
  }
}
```
## Comparing the Results
```java
public class CustomerTest {
  Customer john, steve, pat, david;
  String johnName = "John",
    steveName = "Steve",
    patName = "Pat",
    davidName = "David";
  Customer[] customers;

  @Before
  public void setup() {
    david = ObjectMother
      .customerWithNoRentals(
        davidName);
    john = ObjectMother
      .customerWithOneNewRelease(
        johnName);
    pat = ObjectMother
      .customerWithOneOfEachRentalType(
        patName);
    steve = ObjectMother
      .customerWithOneNewReleaseAndOneRegular(
        steveName);
    customers =
      new Customer[]
      { david, john, steve, pat};
  }
```

```java
  @Test
  public void getName() {
    assertEquals(
      davidName, david.getName());
    assertEquals(
      johnName, john.getName());
    assertEquals(
      steveName, steve.getName());
    assertEquals(
      patName, pat.getName());
  }

  @Test
  public void statement() {
    for (int i=0; i<customers.length; i++) {
      assertEquals(
        expStatement(
          "Rental record for %s\n" +
          "%sAmount owed is %s\n"  +
          "You earned %s frequent " +
          "renter points",
          customers[i],
          rentalInfo(
            "\t", "",
            customers[i].getRentals())),
        customers[i].statement());
    }
  }
```

```java
  @Test
  public void htmlStatement() {
    for (int i=0; i<customers.length; i++) {
      assertEquals(
        expStatement(
          "<h1>Rental record for " +
          "<em>%s</em></h1>\n%s" +
          "<p>Amount owed is <em>%s</em>" +
          "</p>\n<p>You earned <em>%s" +
          " frequent renter points</em></p>",
          customers[i],
          rentalInfo(
            "<p>", "</p>",
            customers[i].getRentals())),
        customers[i].htmlStatement());
    }
  }

  @Test
  (expected=IllegalArgumentException.class)
  public void invalidTitle() {
    ObjectMother
      .customerWithNoRentals("Bob")
      .addRental(
        new Rental(
          new Movie("Crazy, Stupid, Love.",
                    Movie.Type.UNKNOWN),
          4));
  }
```

```java
  public static String rentalInfo(
    String startsWith,
    String endsWith,
    List<Rental> rentals) {
    String result = "";
    for (Rental rental : rentals)
      result += String.format(
        "%s%s\t%s%s\n",
        startsWith,
        rental.getMovie().getTitle(),
        rental.getCharge(),
        endsWith);
    return result;
  }

  public static String expStatement(
    String formatStr,
    Customer customer,
    String rentalInfo) {
    return String.format(
      formatStr,
      customer.getName(),
      rentalInfo,
      customer.getTotalCharge(),
      customer.getTotalPoints());
  }
}
```
```java
public class CustomerTest {
  @Test
  public void getName() {
    assertEquals(
      "John",
      a.customer.w(
        "John").build().getName());
  }

  @Test
  public void noRentalsStatement() {
    assertEquals(
      "Rental record for David\nAmount " +
      "owed is 0.0\nYou earned 0 frequent " +
      "renter points",
      a.customer.w(
        "David").build().statement());
  }

  @Test
  public void oneNewReleaseStatement() {
    assertEquals(
      "Rental record for John\n" +
      "\tGodfather 4 9.0\n" +
      "Amount owed is 9.0\n" +
      "You earned 2 frequent renter points",
      a.customer.w("John").w(
        a.rental.w(
          a.movie.w(
            NEW_RELEASE))).build()
      .statement());
  }
```

```java
  @Test
  public void allRentalTypesStatement() {
    assertEquals(
      "Rental record for Pat\n" +
      "\tGodfather 4 9.0\n" +
      "\tScarface 3.5\n" +
      "\tLion King 1.5\n" +
      "Amount owed is 14.0\n" +
      "You earned 4 frequent renter points",
      a.customer.w("Pat").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(
          a.movie.w("Scarface").w(REGULAR)),
        a.rental.w(
          a.movie.w("Lion King").w(
            CHILDREN))).build().statement());
  }
```

```java
  @Test
  public void
  newReleaseAndRegularStatement() {
    assertEquals(
      "Rental record for Steve\n" +
      "\tGodfather 4 9.0\n" +
      "\tScarface 3.5\n" +
      "Amount owed is 12.5\n" +
      "You earned 3 frequent renter points",
      a.customer.w("Steve").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(
          a.movie.w("Scarface").w(
            REGULAR))).build().statement());
  }

  @Test
  public void noRentalsHtmlStatement() {
    assertEquals(
      "<h1>Rental record for <em>David" +
      "</em></h1>\n<p>Amount owed is <em>" +
      "0.0</em></p>\n<p>You earned <em>0 " +
      "frequent renter points</em></p>",
      a.customer.w(
        "David").build().htmlStatement());
  }
```

```java
  @Test
  public void oneNewReleaseHtmlStatement() {
    assertEquals(
      "<h1>Rental record for <em>John</em>" +
      "</h1>\n<p>Godfather 4 9.0</p>\n" +
      "<p>Amount owed is <em>9.0</em></p>" +
      "\n<p>You earned <em>2 frequent " +
      "renter points</em></p>",
      a.customer.w("John").w(
        a.rental.w(
          a.movie.w(
            NEW_RELEASE))).build()
      .htmlStatement());
  }
```

```java
  @Test
  public void allRentalTypesHtmlStatement() {
    assertEquals(
      "<h1>Rental record for <em>Pat</em>" +
      "</h1>\n<p>Godfather 4 9.0</p>\n" +
      "<p>Scarface 3.5</p>\n<p>Lion King" +
      " 1.5</p>\n<p>Amount owed is <em>" +
      "14.0</em></p>\n<p>You earned <em>" +
      "4 frequent renter points</em></p>",
      a.customer.w("Pat").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(
          a.movie.w("Scarface").w(REGULAR)),
        a.rental.w(
          a.movie.w("Lion King").w(
            CHILDREN))).build()
      .htmlStatement());
  }
```

```java
  @Test
  public void
  newReleaseAndRegularHtmlStatement() {
    assertEquals(
      "<h1>Rental record for <em>Steve" +
      "</em></h1>\n<p>Godfather 4 9.0</p>" +
      "\n<p>Scarface 3.5</p>\n<p>Amount " +
      "owed is <em>12.5</em></p>\n<p>" +
      "You earned <em>3 frequent renter " +
      "points</em></p>",
      a.customer.w("Steve").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(
          a.movie.w("Scarface").w(
            REGULAR))).build()
      .htmlStatement());
  }

  @Test
  (expected=IllegalArgumentException.class)
  public void invalidTitle() {
    a.customer.w(
      a.rental.w(
        a.movie.w(UNKNOWN))).build();
  }
}
```
## Final Thoughts on our Tests
# Motivators
### Validate the System
#### Common motivators that would be a subset of Validate the System
### Code Coverage
### Enable Refactoring
### Document the Behavior of the System
### Your Manager Told You To
### Test Driven Development
#### Common motivators that would be a subset of TDD
### Customer Acceptance
### Ping Pong Pair-Programming
### What Motivates You (or Your Team)
# Types of Tests 
### Strongly Recommended Reference Material
## State Verification 
```java
public class RentalTest {
  @Test
  public void rentalIsStartedIfInStore() {
    Movie movie = a.movie.build();
    Rental rental =
      a.rental.w(movie).build();
    Store store = a.store.w(movie).build();
    rental.start(store);
    assertTrue(rental.isStarted());
    assertEquals(
      0, store.getAvailability(movie));
  }

  @Test
  public void
  rentalDoesNotStartIfNotAvailable() {
    Movie movie = a.movie.build();
    Rental rental = a.rental.build();
    Store store = a.store.build();
    rental.start(store);
    assertFalse(rental.isStarted());
    assertEquals(
      0, store.getAvailability(movie));
  }
}
```
## Behavior Verification 
```java
public class RentalTest {
  @Test
  public void rentalIsStartedIfInStore() {
    Movie movie = a.movie.build();
    Rental rental =
      a.rental.w(movie).build();
    Store store = mock(Store.class);
    when(store.getAvailability(movie))
      .thenReturn(1);
    rental.start(store);
    assertTrue(rental.isStarted());
    verify(store).remove(movie);
  }

  @Test
  public void
  rentalDoesNotStartIfNotAvailable() {
    Rental rental = a.rental.build();
    Store store = mock(Store.class);
    rental.start(store);
    assertFalse(rental.isStarted());
    verify(
      store, never()).remove(
        any(Movie.class));
  }
}
```
### Picking a Side
## Unit Test
## Solitary Unit Test 
## Sociable Unit Test 
## Continuing with Examples From Chapter 1
```java
public class CustomerTest {
  @Test
  public void getName() {
    assertEquals(
      "John",
      a.customer.w(
        "John").build().getName());
  }

  @Test
  public void noRentalsStatement() {
    assertEquals(
      "Rental record for David\nAmount" +
      " owed is 0.0\n" +
      "You earned 0 frequent renter points",
      a.customer.w(
        "David").build().statement());
  }

  @Test
  public void oneNewReleaseStatement() {
    assertEquals(
      "Rental record for John\n" +
      "\tGodfather 4 9.0\n" +
      "Amount owed is 9.0\n" +
      "You earned 2 frequent renter points",
      a.customer.w("John").w(
        a.rental.w(
          a.movie.w(
            NEW_RELEASE))).build()
      .statement());
  }
```

```java
  @Test
  public void allRentalTypesStatement() {
    assertEquals(
      "Rental record for Pat\n" +
      "\tGodfather 4 9.0\n" +
      "\tScarface 3.5\n" +
      "\tLion King 1.5\n" +
      "Amount owed is 14.0\n" +
      "You earned 4 frequent renter points",
      a.customer.w("Pat").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(
          a.movie.w("Scarface").w(REGULAR)),
        a.rental.w(
          a.movie.w(
            "Lion King").w(
              CHILDREN))).build()
      .statement());
  }
```

```java
  @Test
  public void
  newReleaseAndRegularStatement() {
    assertEquals(
      "Rental record for Steve\n" +
      "\tGodfather 4 9.0\n" +
      "\tScarface 3.5\n" +
      "Amount owed is 12.5\n" +
      "You earned 3 frequent renter points",
      a.customer.w("Steve").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(
          a.movie.w("Scarface").w(
            REGULAR))).build()
      .statement());
  }

  @Test
  public void noRentalsHtmlStatement() {
    assertEquals(
      "<h1>Rental record for <em>David" +
      "</em></h1>\n<p>Amount owed is " +
      "<em>0.0</em></p>\n<p>" +
      "You earned <em>0 frequent renter " +
      "points</em></p>",
      a.customer.w(
        "David").build().htmlStatement());
  }
```

```java
  @Test
  public void oneNewReleaseHtmlStatement() {
    assertEquals(
      "<h1>Rental record for <em>John</em>" +
      "</h1>\n<p>Godfather 4 9.0</p>\n" +
      "<p>Amount owed is <em>9.0</em></p>" +
      "\n<p>You earned <em>2 frequent " +
      "renter points</em></p>",
      a.customer.w("John").w(
        a.rental.w(
          a.movie.w(NEW_RELEASE))).build()
      .htmlStatement());
  }

  @Test
  public void allRentalTypesHtmlStatement() {
    assertEquals(
      "<h1>Rental record for <em>Pat</em>" +
      "</h1>\n<p>Godfather 4 9.0</p>\n<p>" +
      "Scarface 3.5</p>\n<p>Lion King 1.5" +
      "</p>\n<p>Amount owed is <em>14.0" +
      "</em></p>\n<p>You earned <em>4 " +
      "frequent renter points</em></p>",
      a.customer.w("Pat").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(a.movie.w("Scarface").w(
                     REGULAR)),
        a.rental.w(a.movie.w("Lion King").w(
                     CHILDREN))).build()
      .htmlStatement());
  }
```

```java
  @Test
  public void
  newReleaseAndRegularHtmlStatement() {
    assertEquals(
      "<h1>Rental record for <em>Steve" +
      "</em></h1>\n<p>Godfather 4 9.0</p>" +
      "\n<p>Scarface 3.5</p>\n<p>Amount " +
      "owed is <em>12.5</em></p>\n<p>You " +
      "earned <em>3 frequent renter points" +
      "</em></p>",
      a.customer.w("Steve").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(a.movie.w("Scarface").w(
                     REGULAR))).build()
      .htmlStatement());
  }

  @Test
  (expected=IllegalArgumentException.class)
  public void invalidTitle() {
    a.customer.w(
      a.rental.w(
        a.movie.w(UNKNOWN))).build();
  }
}
```
```java
public class MovieTest {
  @Test
  (expected=IllegalArgumentException.class)
  public void invalidTitle() {
    a.movie.w(UNKNOWN).build();
  }
}
```
```java
public class CustomerTest {
  @Test
  public void noRentalsStatement() {
    assertEquals(
      "Rental record for Jim\nAmount owed" +
      " is 0.0\n" +
      "You earned 0 frequent renter points",
      a.customer.build().statement());
  }

  @Test
  public void oneRentalStatement() {
    assertEquals(
      "Rental record for Jim\n\tnull\n" +
      "Amount owed is 0.0\n" +
      "You earned 0 frequent renter points",
      a.customer.w(
        
        mock(Rental.class)).build()
      
      .statement());
  }
```

```java
  @Test
  public void twoRentalsStatement() {
    assertEquals(
      "Rental record for Jim\n\t" +
      "null\n\tnull\n" +
      "Amount owed is 0.0\n" +
      "You earned 0 frequent renter points",
      a.customer.w(
        
        mock(Rental.class),
        mock(Rental.class)).build()
      
      .statement());
  }
}
```
```java
public class CustomerTest {
  @Test
  public void noRentalsCharge() {
    assertEquals(
      0.0,
      a.customer.build().getTotalCharge(),
      0);
  }

  @Test
  public void twoRentalsCharge() {
    Rental rental = mock(Rental.class);
    when(rental.getCharge()).thenReturn(2.0);
    assertEquals(
      4.0,
      a.customer.w(
        rental,
        rental).build().getTotalCharge(),
      0);
  }
```

```java
  @Test
  public void threeRentalsCharge() {
    Rental rental = mock(Rental.class);
    when(rental.getCharge()).thenReturn(2.0);
    assertEquals(
      6.0,
      a.customer.w(
        rental,
        rental,
        rental).build().getTotalCharge(),
      0);
  }

  @Test
  public void noRentalsPoints() {
    assertEquals(
      0,
      a.customer.build().getTotalPoints());
  }

  @Test
  public void twoRentalsPoints() {
    Rental rental = mock(Rental.class);
    when(rental.getPoints()).thenReturn(2);
    assertEquals(
      4,
      a.customer.w(
        rental,
        rental).build().getTotalPoints());
  }
```

```java
  @Test
  public void threeRentalsPoints() {
    Rental rental = mock(Rental.class);
    when(rental.getPoints()).thenReturn(2);
    assertEquals(
      6,
      a.customer.w(
        rental,
        rental,
        rental).build().getTotalPoints());
  }
}
```
```java
public class MovieTest {
  @Test
  public void getChargeForChildrens() {
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(1),
      0);
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(2),
      0);
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(3),
      0);
    assertEquals(
      3.0,
      a.movie.w(
        CHILDREN).build().getCharge(4),
      0);
    assertEquals(
      4.5,
      a.movie.w(
        CHILDREN).build().getCharge(5),
      0);
  }
```

```java
  @Test
  public void getChargeForNewRelease() {
    assertEquals(
      3.0,
      a.movie.w(
        NEW_RELEASE).build().getCharge(1),
      0);
    assertEquals(
      6.0,
      a.movie.w(
        NEW_RELEASE).build().getCharge(2),
      0);
    assertEquals(
      9.0,
      a.movie.w(
        NEW_RELEASE).build().getCharge(3),
      0);
  }
```

```java
  @Test
  public void getChargeForRegular() {
    assertEquals(
      2.0,
      a.movie.w(
        REGULAR).build().getCharge(1),
      0);
    assertEquals(
      2.0,
      a.movie.w(
        REGULAR).build().getCharge(2),
      0);
    assertEquals(
      3.5,
      a.movie.w(
        REGULAR).build().getCharge(3),
      0);
    assertEquals(
      5.0,
      a.movie.w(
        REGULAR).build().getCharge(4),
      0);
  }
```

```java
  @Test
  public void getPointsForChildrens() {
    assertEquals(
      1,
      a.movie.w(
        CHILDREN).build().getPoints(1));
    assertEquals(
      1,
      a.movie.w(
        CHILDREN).build().getPoints(2));
  }

  @Test
  public void getPointsForNewRelease() {
    assertEquals(
      1,
      a.movie.w(
        NEW_RELEASE).build().getPoints(1));
    assertEquals(
      2,
      a.movie.w(
        NEW_RELEASE).build().getPoints(2));
    assertEquals(
      2,
      a.movie.w(
        NEW_RELEASE).build().getPoints(3));
  }
```

```java
  @Test
  public void getPointsForRegular() {
    assertEquals(
      1,
      a.movie.w(
        REGULAR).build().getPoints(1));
    assertEquals(
      1,
      a.movie.w(
        REGULAR).build().getPoints(2));
  }
}
```
```java
public class CustomerTest {
  @Test
  public void allRentalTypesStatement() {
    assertEquals(
      "Rental record for Pat\n" +
      "\tGodfather 4 9.0\n" +
      "\tScarface 3.5\n" +
      "\tLion King 1.5\n" +
      "Amount owed is 14.0\n" +
      "You earned 4 frequent renter points",
      a.customer.w("Pat").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(a.movie.w("Scarface").w(
                     REGULAR)),
        a.rental.w(a.movie.w("Lion King").w(
                     CHILDREN)))
      .build().statement());
  }
```

```java
  @Test
  public void allRentalTypesHtmlStatement() {
    assertEquals(
      "<h1>Rental record for " +
      "<em>Pat</em></h1>\n" +
      "<p>Godfather 4 9.0</p>\n" +
      "<p>Scarface 3.5</p>\n" +
      "<p>Lion King 1.5</p>\n" +
      "<p>Amount owed is " +
      "<em>14.0</em></p>\n<p>" +
      "You earned <em>4 frequent " +
      "renter points</em></p>",
      a.customer.w("Pat").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(a.movie.w("Scarface").w(
                     REGULAR)),
        a.rental.w(a.movie.w("Lion King").w(
                     CHILDREN)))
      .build().htmlStatement());
  }
}
```
## Final Thoughts, Again
### The Failure
```java
public class CustomerTest {
  @Test
  public void getName() {
    assertEquals(
      "John",
      a.customer.w(
        "John").build().getName());
  }

  @Test
  public void noRentalsStatement() {
    assertEquals(
      "Rental record for Jim\nAmount owed" +
      " is 0.0\nYou earned 0 frequent " +
      "renter points",
      a.customer.build().statement());
  }

  @Test
  public void oneRentalStatement() {
    assertEquals(
      "Rental record for Jim\n\tnull\n" +
      "Amount owed is 0.0\n" +
      "You earned 0 frequent renter points",
      a.customer.w(
        mock(Rental.class)).build()
      .statement());
  }
```

```java
  @Test
  public void twoRentalsStatement() {
    assertEquals(
      "Rental record for Jim\n\tnull\n" +
      "\tnull\nAmount owed is 0.0\n" +
      "You earned 0 frequent renter points",
      a.customer.w(
        mock(Rental.class),
        mock(Rental.class)).build()
      .statement());
  }

  @Test
  public void noRentalsHtmlStatement() {
    assertEquals(
      "<h1>Rental record for <em>Jim</em>" +
      "</h1>\n<p>Amount owed is <em>0.0" +
      "</em></p>\n<p>You earned <em>0 " +
      "frequent renter points</em></p>",
      a.customer.build().htmlStatement());
  }
```

```java
  @Test
  public void oneRentalHtmlStatement() {
    Rental rental = mock(Rental.class);
    assertEquals(
      "<h1>Rental record for <em>Jim</em>" +
      "</h1>\n<p>null</p>\n<p>Amount owed " +
      "is <em>0.0</em></p>\n<p>You earned " +
      "<em>0 frequent renter points</em>" +
      "</p>",
      a.customer.w(
        mock(Rental.class)).build()
      .htmlStatement());
  }

  @Test
  public void twoRentalsHtmlStatement() {
    assertEquals(
      "<h1>Rental record for <em>Jim</em>" +
      "</h1>\n<p>null</p>\n<p>null</p>\n" +
      "<p>Amount owed is <em>0.0</em></p>" +
      "\n<p>You earned <em>0 frequent" +
      " renter points</em></p>",
      a.customer.w(
        mock(Rental.class),
        mock(Rental.class)).build()
      .htmlStatement());
  }
```

```java
  @Test
  public void noRentalsCharge() {
    assertEquals(
      0.0,
      a.customer.build().getTotalCharge(),
      0);
  }

  @Test
  public void twoRentalsCharge() {
    Rental rental = mock(Rental.class);
    when(rental.getCharge()).thenReturn(2.0);
    assertEquals(
      6.0,
      a.customer.w(
        rental,
        rental).build().getTotalCharge(),
      0);
  }

  @Test
  public void threeRentalsCharge() {
    Rental rental = mock(Rental.class);
    when(rental.getCharge()).thenReturn(2.0);
    assertEquals(
      6.0,
      a.customer.w(
        rental,
        rental,
        rental).build().getTotalCharge(),
      0);
  }
```

```java
  @Test
  public void noRentalsPoints() {
    assertEquals(
      0,
      a.customer.build().getTotalPoints());
  }

  @Test
  public void twoRentalsPoints() {
    Rental rental = mock(Rental.class);
    when(rental.getPoints()).thenReturn(2);
    assertEquals(
      4,
      a.customer.w(
        rental,
        rental).build().getTotalPoints());
  }

  @Test
  public void threeRentalsPoints() {
    Rental rental = mock(Rental.class);
    when(rental.getPoints()).thenReturn(2);
    assertEquals(
      6,
      a.customer.w(
        rental,
        rental,
        rental).build().getTotalPoints());
  }
}
```
```java
public class CustomerTest {
  @Test
  public void allRentalTypesStatement() {
    assertEquals(
      "Rental record for Pat\n" +
      "\tGodfather 4 9.0\n" +
      "\tScarface 3.5\n" +
      "\tLion King 1.5\n" +
      "Amount owed is 14.0\n" +
      "You earned 4 frequent renter points",
      a.customer.w("Pat").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(a.movie.w("Scarface").w(
                     REGULAR)),
        a.rental.w(a.movie.w("Lion King").w(
                     CHILDREN)))
      .build().statement());
  }
```

```java
  @Test
  public void allRentalTypesHtmlStatement() {
    assertEquals(
      "<h1>Rental record for <em>Pat" +
      "</em></h1>\n" +
      "<p>Godfather 4 9.0</p>\n" +
      "<p>Scarface 3.5</p>\n" +
      "<p>Lion King 1.5</p>\n" +
      "<p>Amount owed is " +
      "<em>14.0</em></p>\n<p>" +
      "You earned <em>4 " +
      "frequent renter points</em></p>",
      a.customer.w("Pat").w(
        a.rental.w(a.movie.w(NEW_RELEASE)),
        a.rental.w(a.movie.w("Scarface").w(
                     REGULAR)),
        a.rental.w(a.movie.w("Lion King").w(
                     CHILDREN)))
      .build().htmlStatement());
  }
}
```
```java
public class Customer {

  private String name;
  private List<Rental> rentals
  = new ArrayList<Rental>();

  public Customer(String name) {
    this.name = name;
  }

  public String getName() {
    return name;
  }

  public void addRental(Rental rental) {
    rentals.add(rental);
  }

  public String statement() {
    String result =
      "Rental record for " +
      getName() + "\n";
    for (Rental rental : rentals)
      result +=
        "\t" + rental.getLineItem() + "\n";
    result += "Amount owed is " +
      getTotalCharge() + "\n" +
      "You earned " + getTotalPoints() +
      " frequent renter points";
    return result;
  }
```

```java
  public String htmlStatement() {
    String result =
      "<h1>Rental record for <em>" +
      getName() + "</em></h1>\n";
    for (Rental rental : rentals)
      result +=
        "<p>" + rental.getLineItem() +
        "</p>\n";
    result +=
      "<p>Amount owed is <em>" +
      getTotalCharge() + "</em></p>\n" +
      "<p>You earned <em>" +
      getTotalPoints() +
      " frequent renter points</em></p>";
    return result;

  }

  public double getTotalCharge() {
    double total = 0;
    for (Rental rental : rentals)
      total += rental.getCharge();
    return total;
  }

  public int getTotalPoints() {
    int total = 0;
    for (Rental rental : rentals)
      total += rental.getPoints();
    return total;
  }
}
```
```java
public class MovieTest {
  @Test
  public void getChargeForChildrens() {
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(1),
      0);
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(2),
      0);
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(3),
      0);
    assertEquals(
      3.0,
      a.movie.w(
        CHILDREN).build().getCharge(4),
      0);
    assertEquals(
      4.5,
      a.movie.w(
        CHILDREN).build().getCharge(5),
      0);
  }
```

```java
  @Test
  public void getChargeForNewRelease() {
    assertEquals(
      3.0,
      a.movie.w(
        NEW_RELEASE).build().getCharge(1),
      0);
    assertEquals(
      6.0,
      a.movie.w(
        NEW_RELEASE).build().getCharge(2),
      0);
    assertEquals(
      9.0,
      a.movie.w(
        NEW_RELEASE).build().getCharge(3),
      0);
  }
```

```java
  @Test
  public void getChargeForRegular() {
    assertEquals(
      2.0,
      a.movie.w(
        REGULAR).build().getCharge(1),
      0);
    assertEquals(
      2.0,
      a.movie.w(
        REGULAR).build().getCharge(2),
      0);
    assertEquals(
      3.5,
      a.movie.w(
        REGULAR).build().getCharge(3),
      0);
    assertEquals(
      5.0,
      a.movie.w(
        REGULAR).build().getCharge(4),
      0);
  }
```

```java
  @Test
  public void getPointsForChildrens() {
    assertEquals(
      1,
      a.movie.w(
        CHILDREN).build().getPoints(1));
    assertEquals(
      1,
      a.movie.w(
        CHILDREN).build().getPoints(2));
  }

  @Test
  public void getPointsForNewRelease() {
    assertEquals(
      1,
      a.movie.w(
        NEW_RELEASE).build().getPoints(1));
    assertEquals(
      2,
      a.movie.w(
        NEW_RELEASE).build().getPoints(2));
    assertEquals(
      2,
      a.movie.w(
        NEW_RELEASE).build().getPoints(3));
  }
```

```java
  @Test
  public void getPointsForRegular() {
    assertEquals(
      1,
      a.movie.w(
        REGULAR).build().getPoints(1));
    assertEquals(
      1,
      a.movie.w(
        REGULAR).build().getPoints(2));
  }

  @Test
  (expected=IllegalArgumentException.class)
  public void invalidTitle() {
    a.movie.w(UNKNOWN).build();
  }
}
```
```java
public class Movie {

  public enum Type {
    REGULAR, NEW_RELEASE, CHILDREN, UNKNOWN;
  }

  private String title;
  Price price;

  public Movie(
    String title, Movie.Type priceCode) {
    this.title = title;
    setPriceCode(priceCode);
  }

  public String getTitle() {
    return title;
  }
```

```java
  private void setPriceCode(
    Movie.Type priceCode) {
    switch (priceCode) {
    case CHILDREN:
      price = new ChildrensPrice();
      break;
    case NEW_RELEASE:
      price = new NewReleasePrice();
      break;
    case REGULAR:
      price = new RegularPrice();
      break;
    default:
      throw new IllegalArgumentException(
        "invalid price code");
    }
  }

  public double getCharge(int daysRented) {
    return price.getCharge(daysRented);
  }

  public int getPoints(int daysRented) {
    return price.getPoints(daysRented);
  }
}
```
```java
public class RentalTest {
  @Test
  public void
  isStartedIfInStoreStateBased() {
    Movie movie = a.movie.build();
    Rental rental =
      a.rental.w(movie).build();
    Store store = a.store.w(movie).build();
    rental.start(store);
    assertTrue(rental.isStarted());
    assertEquals(
      0, store.getAvailability(movie));
  }

  @Test
  public void
  doesNotStartIfNotAvailableStateBased() {
    Movie movie = a.movie.build();
    Rental rental = a.rental.build();
    Store store = a.store.build();
    rental.start(store);
    assertFalse(rental.isStarted());
    assertEquals(
      0, store.getAvailability(movie));
  }
```

```java
  @Test
  public void
  isStartedIfInStoreInteractionBased() {
    Movie movie = a.movie.build();
    Rental rental =
      a.rental.w(movie).build();
    Store store = mock(Store.class);
    when(store.getAvailability(movie))
      .thenReturn(1);
    rental.start(store);
    assertTrue(rental.isStarted());
    verify(store).remove(movie);
  }

  @Test
  public void
  notStartedIfUnavailableInteractionBased() {
    Rental rental = a.rental.build();
    Store store = mock(Store.class);
    rental.start(store);
    assertFalse(rental.isStarted());
    verify(
      store, never()).remove(
        any(Movie.class));
  }
}
```
```java
public class Rental {

  Movie movie;
  private int daysRented;
  private boolean started;

  public Rental(
    Movie movie, int daysRented) {
    this.movie = movie;
    this.daysRented = daysRented;
  }

  public Movie getMovie() {
    return movie;
  }

  public int getDaysRented() {
    return daysRented;
  }

  public double getCharge() {
    return movie.getCharge(daysRented);
  }

  public int getPoints() {
    return movie.getPoints(daysRented);
  }

  public String getLineItem() {
    return
      movie.getTitle() + " " + getCharge();
  }
```

```java
  public boolean isStarted() {
    return started;
  }

  public void start(Store store) {
    if (store.getAvailability(movie) > 0) {
      store.remove(movie);
      this.started = true;
    }
  }
}
```
```java
public class Store {
  private Map<Movie, Integer> movies;

  public Store(Map<Movie, Integer> movies) {
    this.movies = movies;
  }

  public int getAvailability(Movie movie) {
    if (null == movies.get(movie))
      return 0;
    return movies.get(movie);
  }

  public boolean getAvailability(
    Movie movie, int quantity) {
    if (null == movies.get(movie))
      return false;
    return movies.get(movie) >= quantity;
  }

  public void remove(Movie movie) {
    if (null == movies.get(movie))
      return;
    Integer count = movies.get(movie);
    movies.put(movie, --count);
  }
}
```
# Improving Assertions 
## One Assertion Per Test 
```java
public class MovieTest {
  @Test
  public void getChargeForChildrens() {
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(1),
      0);
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(2),
      0);
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(3),
      0);
    assertEquals(
      3.0,
      a.movie.w(
        CHILDREN).build().getCharge(4),
      0);
    assertEquals(
      4.5,
      a.movie.w(
        CHILDREN).build().getCharge(5),
      0);
  }
}
```
### Failure
```java
public class ChildrensPrice extends Price {

  @Override
  public double getCharge(int daysRented) {
    double amount = 1.5;
    
    if (daysRented > 2) // *was 3*
      amount += (daysRented - 2) * 1.5;
      
    return amount;
  }
}
```
### Test Naming
### The Solution
```java
public class MovieTest {
  @Test
  public void getChargeForChildrens1Day() {
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(1),
      0);
  }

  @Test
  public void getChargeForChildrens2Day() {
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(2),
      0);
  }

  @Test
  public void getChargeForChildrens3Day() {
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(3),
      0);
  }
```

```java
  @Test
  public void getChargeForChildrens4Day() {
    assertEquals(
      3.0,
      a.movie.w(
        CHILDREN).build().getCharge(4),
      0);
  }

  @Test
  public void getChargeForChildrens5Day() {
    assertEquals(
      4.5,
      a.movie.w(
        CHILDREN).build().getCharge(5),
      0);
  }
}
```
### Applying One Assertion Per Test To Behavior Verification Tests
```java
public class RentalTest {
  @Test
  public void rentalIsStartedIfInStore() {
    Movie movie = mock(Movie.class);
    Rental rental =
      a.rental.w(movie).build();
    Store store = mock(Store.class);
    when(store.getAvailability(movie))
      .thenReturn(1);
    rental.start(store);
    assertTrue(rental.isStarted());
    verify(store).remove(movie);
  }
}
```
```java
public class RentalTest {
  @Test
  public void rentalIsStartedIfInStore() {
    Movie movie = mock(Movie.class);
    Rental rental =
      a.rental.w(movie).build();
    Store store = mock(Store.class);
    
    when(store.getAvailability(movie, 1))
      .thenReturn(true);
    
    rental.start(store);
    assertTrue(rental.isStarted());
    verify(store).remove(movie);
  }
}
```
```java
public class RentalTest {
  @Test
  public void rentalIsStartedIfInStore() {
    Rental rental = a.rental.build();
    Store store = mock(Store.class);
    when(store
         .getAvailability(any(Movie.class)))
      .thenReturn(1);
    rental.start(store);
    assertTrue(rental.isStarted());
  }
}
```
```java
public class StoreTest {
  @Test
  public void storeWithNoAvailability() {
    Store store = a.store.build();
    assertEquals(
      0,
      store.getAvailability(
        mock(Movie.class)));
  }

  @Test
  public void storeWithAvailability() {
    Movie movie = mock(Movie.class);
    Store store =
      a.store.w(movie, movie).build();
    assertEquals(
      2, store.getAvailability(movie));
  }

  @Test
  public void
  storeWithRemovedAvailability() {
    Movie movie = mock(Movie.class);
    Store store =
      a.store.w(movie, movie).build();
    store.remove(movie);
    assertEquals(
      1, store.getAvailability(movie));
  }
}
```
```java
public class RentalTest {
  @Test
  public void
  storeAvailabilityIsModifiedOnRental() {
    Movie movie = a.movie.build();
    Rental rental =
      a.rental.w(movie).build();
    Store store =
      a.store.w(movie, movie).build();
    rental.start(store);
    a.rental.build().start(store);
    assertEquals(
      1, store.getAvailability(movie));
  }
}
```
### Thoughts On The Result
## Implementation Overspecification 
```java
public class Customer {

  private String name;
  private List<Rental> rentals =
    new ArrayList<Rental>();

  public Customer(String name) {
    this.name = name;
  }

  public void addRental(Rental rental) {
    rentals.add(rental);
  }

  
  public String recentRentals() {
    String result = "Recent rentals:";
    for (int i=0;
         i < rentals.size() && i < 3;
         i++) {
      result += "\n" +
        rentals.get(i).getMovie(
          true).getTitle(
            "%s starring %s %s", 2);
    }
    return result;
  }
  
}
```
```java
public class CustomerTest {
  @Test
  public void recentRentalsWith2Rentals() {
    Movie godfather = mock(Movie.class);
    when(godfather
         .getTitle("%s starring %s %s", 2))
      .thenReturn("Godfather 4");
    Rental godfatherRental =
      mock(Rental.class);
    when(godfatherRental.getMovie(true))
      .thenReturn(godfather);
    Movie lionKing = mock(Movie.class);
    when(lionKing
         .getTitle("%s starring %s %s", 2))
      .thenReturn("Lion King");
    Rental lionKingRental =
      mock(Rental.class);
    when(lionKingRental.getMovie(true))
      .thenReturn(lionKing);

    assertEquals(
      "Recent rentals:\nGodfather 4\n" +
      "Lion King",
      a.customer.w(
        godfatherRental, lionKingRental)
      .build().recentRentals());
  }
```

```java
  @Test
  public void recentRentalsWith3Rentals() {
    // same structure as above, with
    // 8 more lines of mocking code,
    // 25% longer expected value, and
    // 2 lines of adding rentals to customer
  }

  @Test
  public void recentRentalsWith4Rentals() {
    // same structure as above, with
    // 16 more lines of mocking code,
    // 25% longer expected value, and
    // 2 lines of adding rentals to customer
  }
}
```
### Flexible Argument Matchers
```java
public class CustomerTest {
  @Test
  public void recentRentalsWith2Rentals() {
    Movie godfather = mock(Movie.class);
    when(
      godfather.getTitle(
        
        anyString(), anyInt()))
      
      .thenReturn("Godfather 4");
    Rental godfatherRental =
      mock(Rental.class);
    when(
      
      godfatherRental.getMovie(anyBoolean()))
      
      .thenReturn(godfather);
    Movie lionKing = mock(Movie.class);
    when(
      lionKing.getTitle(
        
        anyString(), anyInt()))
      
      .thenReturn("Lion King");
    Rental lionKingRental =
      mock(Rental.class);
    when(
      
      lionKingRental.getMovie(anyBoolean()))
      
      .thenReturn(lionKing);

    assertEquals(
      "Recent rentals:\nGodfather 4\n" +
      "Lion King",
      a.customer.w(
        godfatherRental, lionKingRental)
      .build().recentRentals());
  }
}
```
### Default Return Values
```java
public class CustomerTest {
  @Test
  public void recentRentalsWith2Rentals() {
    Movie godfather = mock(Movie.class);
    
    Rental godfatherRental =
      mock(Rental.class);
    
    when(
      godfatherRental.getMovie(anyBoolean()))
      .thenReturn(godfather);
    Movie lionKing = mock(Movie.class);
    
    Rental lionKingRental =
      mock(Rental.class);
    
    when(
      lionKingRental.getMovie(anyBoolean()))
      .thenReturn(lionKing);

    assertEquals(
      "Recent rentals:\nnull\nnull",
      a.customer.w(
        godfatherRental, lionKingRental)
      .build().recentRentals());
  }
}
```
```java
public class CustomerTest {
  @Test
  public void recentRentalsWith2Rentals() {
    Movie movie = mock(Movie.class);
    Rental rental = mock(Rental.class);
    when(rental.getMovie(anyBoolean()))
      .thenReturn(movie);
    assertEquals(
      "Recent rentals:\nnull\nnull",
      a.customer.w(rental, rental).build()
      .recentRentals());
  }
}
```
### Law of Demeter
```java
public class CustomerTest {
  @Test
  public void recentRentalsWith2Rentals() {
    Rental rental = mock(Rental.class);
    assertEquals(
      "Recent rentals:\nnull\nnull",
      a.customer.w(rental, rental).build()
      .recentRentals());
  }
}
```
```java
public class Customer {

  private String name;
  private List<Rental> rentals =
    new ArrayList<Rental>();

  public Customer(String name) {
    this.name = name;
  }

  public void addRental(Rental rental) {
    rentals.add(rental);
  }

  public String recentRentals() {
    String result = "Recent rentals:";
    for (int i=0;
         i < rentals.size() && i < 3;
         i++) {
      result +=
        
        "\n" + rentals.get(i).getTitle();
      
    }
    return result;
  }
}
```
```java
public class CustomerTest {
  @Test
  public void recentRentals0Rentals() {
    assertEquals(
      "Recent rentals:",
      a.customer.build().recentRentals());
  }

  @Test
  public void recentRentals1Rental() {
    assertEquals(
      "Recent rentals:\nnull",
      a.customer.w(
        mock(Rental.class)).build()
      .recentRentals());
  }

  @Test
  public void recentRentals2Rental() {
    assertEquals(
      "Recent rentals:\nnull\nnull",
      a.customer.w(
        mock(Rental.class),
        mock(Rental.class)).build()
      .recentRentals());
  }
```

```java
  @Test
  public void recentRentals3Rental() {
    assertEquals(
      "Recent rentals:\nnull\nnull\nnull",
      a.customer.w(
        mock(Rental.class),
        mock(Rental.class),
        mock(Rental.class)).build()
      .recentRentals());
  }

  @Test
  public void recentRentals4Rental() {
    assertEquals(
      "Recent rentals:\nnull\nnull\nnull",
      a.customer.w(
        mock(Rental.class),
        mock(Rental.class),
        mock(Rental.class),
        mock(Rental.class)).build()
      .recentRentals());
  }
}
```
### Get Sociable
```java
public class CustomerTest {
  @Test
  public void
  recentRentalsWith3OrderedRentals() {
    assertEquals(
      "Recent rentals:"+
      "\nGodfather 4\nLion King\nScarface",
      a.customer.w(
        a.rental.w(a.movie.w("Godfather 4")),
        a.rental.w(a.movie.w("Lion King")),
        a.rental.w(a.movie.w("Scarface")),
        a.rental.w(a.movie.w("Notebook")))
      .build().recentRentals());
  }
}
```
### Comparison
#### The (Previously Unwritten) Original Test with Four Rentals
```java
public class CustomerTest {
  @Test
  public void recentRentalsWith4Rentals() {
    Movie godfather = mock(Movie.class);
    when(godfather
         .getTitle("%s starring %s %s", 2))
      .thenReturn("Godfather 4");
    Rental godfatherRental =
      mock(Rental.class);
    when(godfatherRental.getMovie(true))
      .thenReturn(godfather);
    Movie lionKing = mock(Movie.class);
    when(lionKing
         .getTitle("%s starring %s %s", 2))
      .thenReturn("Lion King");
    Rental lionKingRental =
      mock(Rental.class);
    when(lionKingRental.getMovie(true))
      .thenReturn(lionKing);
    Movie scarface = mock(Movie.class);
    when(scarface
         .getTitle("%s starring %s %s", 2))
      .thenReturn("Scarface");
    Rental scarfaceRental =
      mock(Rental.class);
    when(scarfaceRental.getMovie(true))
      .thenReturn(scarface);
    Movie notebook = mock(Movie.class);
    when(notebook
         .getTitle("%s starring %s %s", 2))
      .thenReturn("Notebook");
    Rental notebookRental =
      mock(Rental.class);
    when(notebookRental.getMovie(true))
      .thenReturn(notebook);

    assertEquals(
      "Recent rentals:"+
      "\nGodfather 4\nLion King" +
      "\nScarface",
      a.customer.w(
        godfatherRental, lionKingRental,
        scarfaceRental, notebookRental)
      .build().recentRentals());
  }
}
```
#### The Sociable Unit Test and The Sparsely Specified Solitary Unit Tests
```java
public class CustomerTest {
  @Test
  public void
  recentRentalsWith3OrderedRentals() {
    assertEquals(
      "Recent rentals:"+
      "\nGodfather 4\nLion King\nScarface",
      a.customer.w(
        a.rental.w(a.movie.w("Godfather 4")),
        a.rental.w(a.movie.w("Lion King")),
        a.rental.w(a.movie.w("Scarface")),
        a.rental.w(a.movie.w("Notebook")))
      .build().recentRentals());
  }
}
```
```java
public class CustomerTest {
  @Test
  public void recentRentals0Rentals() {
    assertEquals(
      "Recent rentals:",
      a.customer.build().recentRentals());
  }

  @Test
  public void recentRentals1Rental() {
    assertEquals(
      "Recent rentals:\nnull",
      a.customer.w(
        mock(Rental.class)).build()
      .recentRentals());
  }

  @Test
  public void recentRentals2Rental() {
    assertEquals(
      "Recent rentals:\nnull\nnull",
      a.customer.w(
        mock(Rental.class),
        mock(Rental.class)).build()
      .recentRentals());
  }
```

```java
  @Test
  public void recentRentals3Rental() {
    assertEquals(
      "Recent rentals:\nnull\nnull\nnull",
      a.customer.w(
        mock(Rental.class),
        mock(Rental.class),
        mock(Rental.class)).build()
      .recentRentals());
  }

  @Test
  public void recentRentals4Rental() {
    assertEquals(
      "Recent rentals:\nnull\nnull\nnull",
      a.customer.w(
        mock(Rental.class),
        mock(Rental.class),
        mock(Rental.class),
        mock(Rental.class)).build()
      .recentRentals());
  }
}
```
## Assert Last 
### Expect Exceptions via Try/Catch
```java
public class MovieTest {
  @Test
  (expected=IllegalArgumentException.class)
  public void invalidTitle() {
    a.movie.w(UNKNOWN).build();
  }
}
```
```java
public class MovieTest {
  @Test
  public void invalidTitle() {
    try {
      a.movie.w(UNKNOWN).build();
      fail();
    } catch (Exception ex) {
      assertEquals(
        IllegalArgumentException.class,
        ex.getClass());
    }
  }
}
```
```java
public class MovieTest {
  @Test
  public void invalidTitle() {
    Exception e = null;
    try {
      a.movie.w(UNKNOWN).build();
    } catch (Exception ex) {
      e = ex;
    }
    assertEquals(
      IllegalArgumentException.class,
      e.getClass());
  }
}
```
### Assert Throws
```java
public class MovieTest {
  @Test
  public void invalidTitle() {
    Runnable runnable = new Runnable() {
        public void run() {
          a.movie.w(UNKNOWN).build();
        }
      };
    assertThrows(
      IllegalArgumentException.class,
      runnable);
  }

  public void assertThrows(
    Class ex, Runnable runnable) {
    Exception exThrown = null;
    try {
      runnable.run();
    } catch (Exception exThrownActual) {
      exThrown = exThrownActual;
    }
    if (null == exThrown)
      fail("No exception thrown");
    else
      assertEquals(ex, exThrown.getClass());
  }
}
```
### Mock Verification
### Comparison
```java
public class MovieTest {
  Mockery context = new Mockery();

  @Test
  public void getPointsForDays() {
    Movie movie = a.movie.build();
    assertEquals(1, movie.getPoints(2));
    assertEquals(1, movie.getPoints(3));
  }

  @Test
  (expected=IllegalArgumentException.class)
  public void invalidTitle() {
    a.movie.w(UNKNOWN).build();
  }

  @Test
  public void getPriceFromPriceInstance() {
    final Price price =
      context.mock(Price.class);
    Movie movie = a.movie.build();
    movie.setPrice(price);

    context.checking(new Expectations() {{
      oneOf(price).getCharge(3);
    }});

    movie.getCharge(3);
    context.assertIsSatisfied();
  }
}
```
```java
public class MovieTest {
  @Test
  public void getPoints2Days() {
    assertEquals(
      2, a.movie.build().getPoints(2));
  }

  @Test
  public void getPoints3Days() {
    assertEquals(
      2, a.movie.build().getPoints(3));
  }

  @Test
  public void invalidTitle() {
    Runnable runnable = new Runnable() {
        public void run() {
          a.movie.w(UNKNOWN).build();
        }
      };
    assertThrows(
      IllegalArgumentException.class,
      runnable);
  }
```

```java
  @Test
  public void getPriceFromPriceInstance() {
    Price price = mock(Price.class);
    Movie movie = a.movie.build();
    movie.setPrice(price);
    movie.getCharge(3);
    verify(price).getCharge(3);
  }
}
```
## Expect Literals 
```java
public class RegularPriceTest {
  @Test
  public void chargeWithStaticVal() {
    assertEquals(
      basePrice,
      a.regularPrice.build().getCharge(2),
      0);
  }

  @Test
  public void chargeWithLocalVal() {
    int daysRented = 4;
    double charge =
      basePrice + (
        daysRented - 2) * multiplier;
    assertEquals(
      charge,
      a.regularPrice.build().getCharge(
        daysRented),
      0);
  }

  @Test
  public void chargeWithLiteral() {
    assertEquals(
      5.0,
      a.regularPrice.build().getCharge(4),
      0);
  }
}
```
### Value Objects vs Expect Literals
```java
public class MovieTest {
  @Test
  public void compareDates() {
    Movie godfather =
      a.movie.w(
        new Date(70261200000L)).build();
    assertEquals(
      "1972-03-24",
      new SimpleDateFormat(
        "yyyy-MM-dd").format(
          godfather.releaseDate()));
  }
}
```
### Comparison
```java
public class CustomerTest {
  @Test
  public void statementFor1Rental() {
    Rental rental = mock(Rental.class);
    Customer customer =
      a.customer.w(rental).build();

    assertEquals(
      expStatement(
        "Rental record for %s\n%sAmount " +
        "owed is %s\n" +
        "You earned %s frequent " +
        "renter points",
        customer,
        rentalInfo(
          "\t", "", new Rental[] {rental})),
      customer.statement());
  }
```

```java
  @Test
  public void statementFor2Rentals() {
    Rental godfather = mock(Rental.class);
    Rental scarface = mock(Rental.class);
    Customer customer =
      a.customer.w(
        godfather, scarface).build();

    assertEquals(
      expStatement(
        "Rental record for %s\n%sAmount " +
        "owed is %s\n" +
        "You earned %s frequent " +
        "renter points",
        customer,
        rentalInfo(
          "\t", "", new Rental[] {
            godfather, scarface})),
      customer.statement());
  }
```

```java
  public static String rentalInfo(
    String startsWith,
    String endsWith,
    Rental[] rentals) {
    String result = "";
    for (Rental rental : rentals)
      result += String.format(
        "%s%s%s\n",
        startsWith,
        rental.getLineItem(),
        endsWith);
    return result;
  }

  public static String expStatement(
    String formatStr,
    Customer customer,
    String rentalInfo) {
    return String.format(
      formatStr,
      customer.getName(),
      rentalInfo,
      customer.getTotalCharge(),
      customer.getTotalPoints());
  }
}
```
```java
public class CustomerTest {
  @Test
  public void statementFor1Rental() {
    Customer customer =
      a.customer.w(
        mock(Rental.class)).build();

    assertEquals(
      "Rental record for Jim\n" +
      "\tnull\n" +
      "Amount owed is 0.0\n" +
      "You earned 0 frequent renter points",
      customer.statement());
  }

  @Test
  public void statementFor2Rentals() {
    Customer customer =
      a.customer.w(
        mock(Rental.class),
        mock(Rental.class)).build();

    assertEquals(
      "Rental record for Jim\n"+
      "\tnull\n" +
      "\tnull\n" +
      "Amount owed is 0.0\n" +
      "You earned 0 frequent renter points",
      customer.statement());
  }
}
```
## Negative Testing 
```java
public class RentalTest {
  @Test
  public void
  storeMockNeverReceivesRemove() {
    Movie movie = mock(Movie.class);
    Rental rental =
      a.rental.w(movie).build();
    Store store = mock(Store.class);
    when(
      store.getAvailability(
        any(Movie.class)))
      .thenReturn(0);
    rental.start(store);
    verify(store, never()).remove(movie);
  }

  @Test
  public void failOnStoreRemove() {
    Movie movie = mock(Movie.class);
    Rental rental =
      a.rental.w(movie).build();
    Store store = new Store(
      new HashMap<Movie,Integer>()) {
        public void remove(Movie movie) {
          fail();
        }
      };
    rental.start(store);
  }
```

```java
  @Test
  public void storeShouldNeverRemove() {
    final boolean[] removeCalled = { false };
    Movie movie = mock(Movie.class);
    Rental rental =
      a.rental.w(movie).build();
    Store store = new Store(
      new HashMap<Movie,Integer>()) {
        public void remove(Movie movie) {
          removeCalled[0] = true;
        }
      };
    rental.start(store);
    assertFalse(removeCalled[0]);
  }
}
```
### Strict Mocking
```java
public class RentalTest {
  @Test
  public void verifyStoreInteractions() {
    Movie movie = mock(Movie.class);
    Rental rental =
      a.rental.w(movie).build();
    Store store = mock(Store.class);
    rental.start(store);
    verify(store).getAvailability(movie);
    verifyNoMoreInteractions(store);
  }
}
```
### Just Be Sociable
```java
public class RentalTest {
  @Test
  public void
  storeAvailabilityIsModifiedOnRental() {
    Movie movie = a.movie.build();
    Rental rental =
      a.rental.w(movie).build();
    Store store =
      a.store.w(movie, movie).build();
    rental.start(store);
    a.rental.build().start(store);
    assertEquals(
      1, store.getAvailability(movie));
  }

  
  @Test
  public void
  storeAvailabilityIsUnmodified() {
    Movie movie = a.movie.build();
    Rental rental =
      a.rental.w(movie).build();
    Store store = a.store.build();
    rental.start(store);
    assertEquals(
      0, store.getAvailability(movie));
  }
  
}
```
## Hamcrest
# Improving Test Cases
## Too Much Magic
### Self-shunt
```java
public class RentalTest extends Store {
  public static Movie movie =
    mock(Movie.class);
  private boolean removeCalled;

  public RentalTest() {
    super(new HashMap<Movie, Integer>() {{
          this.put(movie, 2);
        }});
  }

  @Test
  public void removeIsCalled() {
    Rental rental =
      a.rental.w(movie).build();
    rental.start(this);
    assertEquals(true, removeCalled);
  }

  public void remove(Movie movie) {
    super.remove(movie);
    removeCalled = true;
  }
}
```
### Exceptional Success
```java
public class RentalTest {
  @Test(expected=RuntimeException.class)
  public void removeIsCalled() {
    final Movie movie = mock(Movie.class);
    Rental rental =
      a.rental.w(movie).build();
    HashMap<Movie, Integer> movieMap =
      new HashMap<Movie, Integer>() {{
        this.put(movie, 2);
      }};
    Store store =
      new Store(movieMap) {
        public void remove(Movie movie) {
          throw new
            RuntimeException("success");
        }
      };
    rental.start(store);
  }
}
```
## Inline Setup 
```java
public class CustomerTest {
  Rental godfatherRental;
  Rental lionKingRental;
  Rental scarfaceRental;
  Rental notebookRental;
  Customer twoRentals;
  Customer fourRentals;

  
  @Before
  public void init() {
    godfatherRental = mock(Rental.class);
    when(godfatherRental.getTitle())
      .thenReturn("Godfather 4");
    when(godfatherRental.getCharge())
      .thenReturn(3.0);
    when(godfatherRental.getPoints())
      .thenReturn(2);
    lionKingRental = mock(Rental.class);
    when(lionKingRental.getTitle())
      .thenReturn("Lion King");
    when(lionKingRental.getCharge())
      .thenReturn(2.0);
    when(lionKingRental.getPoints())
      .thenReturn(1);
    scarfaceRental = mock(Rental.class);
    when(scarfaceRental.getTitle())
      .thenReturn("Scarface");
    when(scarfaceRental.getCharge())
      .thenReturn(1.0);
    when(scarfaceRental.getPoints())
      .thenReturn(1);
    notebookRental = mock(Rental.class);
    when(notebookRental.getTitle())
      .thenReturn("Notebook");
    when(notebookRental.getCharge())
      .thenReturn(6.0);
    when(notebookRental.getPoints())
      .thenReturn(1);

    twoRentals =
      a.customer.w(
        godfatherRental, lionKingRental)
      .build();

    fourRentals =
      a.customer.w(
        godfatherRental, lionKingRental,
        scarfaceRental, notebookRental)
      .build();
  }
```

```java
  

  @Test
  public void recentRentalsWith2Rentals() {
    assertEquals(
      "Recent rentals:"+
      "\nGodfather 4\nLion King",
      twoRentals.recentRentals());
  }

  @Test
  public void recentRentalsWith4Rentals() {
    assertEquals(
      "Recent rentals:"+
      "\nGodfather 4\nLion King\nScarface",
      fourRentals.recentRentals());
  }

  
  @Test
  public void totalChargeWith2Rentals() {
    assertEquals(
      5.0,
      twoRentals.getTotalCharge(),
      0);
  }
  

  @Test
  public void totalChargeWith4Rentals() {
    assertEquals(
      12.0,
      fourRentals.getTotalCharge(),
      0);
  }
```

```java
  @Test
  public void totalPointsWith2Rentals() {
    assertEquals(
      3,
      twoRentals.getTotalPoints());
  }

  @Test
  public void totalPointsWith4Rentals() {
    assertEquals(
      5,
      fourRentals.getTotalPoints());
  }

  
  @Test
  public void getName() {
    assertEquals(
      "Jim", twoRentals.getName());
  }
  
}
```
### Similar Creation and Action
### Obviousness
### Setup As An Optimization
### Comparison
```java
public class CustomerTest {
  Rental godfatherRental;
  Rental lionKingRental;
  Rental scarfaceRental;
  Rental notebookRental;
  Customer twoRentals;
  Customer fourRentals;

  
  @Before
  public void init() {
    godfatherRental = mock(Rental.class);
    when(godfatherRental.getTitle())
      .thenReturn("Godfather 4");
    when(godfatherRental.getCharge())
      .thenReturn(3.0);
    when(godfatherRental.getPoints())
      .thenReturn(2);
    lionKingRental = mock(Rental.class);
    when(lionKingRental.getTitle())
      .thenReturn("Lion King");
    when(lionKingRental.getCharge())
      .thenReturn(2.0);
    when(lionKingRental.getPoints())
      .thenReturn(1);
    scarfaceRental = mock(Rental.class);
    when(scarfaceRental.getTitle())
      .thenReturn("Scarface");
    when(scarfaceRental.getCharge())
      .thenReturn(1.0);
    when(scarfaceRental.getPoints())
      .thenReturn(1);
    notebookRental = mock(Rental.class);
    when(notebookRental.getTitle())
      .thenReturn("Notebook");
    when(notebookRental.getCharge())
      .thenReturn(6.0);
    when(notebookRental.getPoints())
      .thenReturn(1);

    twoRentals =
      a.customer.w(
        godfatherRental, lionKingRental)
      .build();

    fourRentals =
      a.customer.w(
        godfatherRental, lionKingRental,
        scarfaceRental, notebookRental)
      .build();
  }
```

```java
  

  @Test
  public void recentRentalsWith2Rentals() {
    assertEquals(
      "Recent rentals:"+
      "\nGodfather 4\nLion King",
      twoRentals.recentRentals());
  }

  @Test
  public void recentRentalsWith4Rentals() {
    assertEquals(
      "Recent rentals:"+
      "\nGodfather 4\nLion King\nScarface",
      fourRentals.recentRentals());
  }

  
  @Test
  public void totalChargeWith2Rentals() {
    assertEquals(
      5.0,
      twoRentals.getTotalCharge(),
      0);
  }
  

  @Test
  public void totalChargeWith4Rentals() {
    assertEquals(
      12.0,
      fourRentals.getTotalCharge(),
      0);
  }
```

```java
  @Test
  public void totalPointsWith2Rentals() {
    assertEquals(
      3,
      twoRentals.getTotalPoints());
  }

  @Test
  public void totalPointsWith4Rentals() {
    assertEquals(
      5,
      fourRentals.getTotalPoints());
  }

  
  @Test
  public void getName() {
    assertEquals(
      "Jim", twoRentals.getName());
  }
  
}
```
```java
public class CustomerTest {
  @Test
  public void recentRentalsWith2Rentals() {
    Rental godfatherRental =
      mock(Rental.class);
    when(godfatherRental.getTitle())
      .thenReturn("Godfather 4");
    Rental lionKingRental =
      mock(Rental.class);
    when(lionKingRental.getTitle())
      .thenReturn("Lion King");
    assertEquals(
      "Recent rentals:"+
      "\nGodfather 4\nLion King",
      a.customer.w(
        godfatherRental, lionKingRental)
      .build().recentRentals());
  }
```

```java
  @Test
  public void recentRentalsWith4Rentals() {
    Rental godfatherRental =
      mock(Rental.class);
    when(godfatherRental.getTitle())
      .thenReturn("Godfather 4");
    Rental lionKingRental =
      mock(Rental.class);
    when(lionKingRental.getTitle())
      .thenReturn("Lion King");
    Rental scarfaceRental =
      mock(Rental.class);
    when(scarfaceRental.getTitle())
      .thenReturn("Scarface");
    Rental notebookRental =
      mock(Rental.class);
    when(notebookRental.getTitle())
      .thenReturn("Notebook");
    assertEquals(
      "Recent rentals:"+
      "\nGodfather 4\nLion King\nScarface",
      a.customer.w(
        godfatherRental, lionKingRental,
        scarfaceRental, notebookRental)
      .build().recentRentals());
  }
```

```java
  
  @Test
  public void totalChargeWith2Rentals() {
    Rental godfatherRental =
      mock(Rental.class);
    when(godfatherRental.getCharge())
      .thenReturn(3.0);
    Rental lionKingRental =
      mock(Rental.class);
    when(lionKingRental.getCharge())
      .thenReturn(2.0);
    assertEquals(
      5.0,
      a.customer.w(
        godfatherRental, lionKingRental)
      .build().getTotalCharge(),
      0);
  }
```

```java
  

  @Test
  public void totalChargeWith4Rentals() {
    Rental godfatherRental =
      mock(Rental.class);
    when(godfatherRental.getCharge())
      .thenReturn(3.0);
    Rental lionKingRental =
      mock(Rental.class);
    when(lionKingRental.getCharge())
      .thenReturn(2.0);
    Rental scarfaceRental =
      mock(Rental.class);
    when(scarfaceRental.getCharge())
      .thenReturn(1.0);
    Rental notebookRental =
      mock(Rental.class);
    when(notebookRental.getCharge())
      .thenReturn(6.0);
    assertEquals(
      12.0,
      a.customer.w(
        godfatherRental, lionKingRental,
        scarfaceRental, notebookRental)
      .build().getTotalCharge(),
      0);
  }
```

```java
  @Test
  public void totalPointsWith2Rentals() {
    Rental godfatherRental =
      mock(Rental.class);
    when(godfatherRental.getPoints())
      .thenReturn(2);
    Rental lionKingRental =
      mock(Rental.class);
    when(lionKingRental.getPoints())
      .thenReturn(1);
    assertEquals(
      3,
      a.customer.w(
        godfatherRental, lionKingRental)
      .build().getTotalPoints());
  }
```

```java
  @Test
  public void totalPointsWith4Rentals() {
    Rental godfatherRental =
      mock(Rental.class);
    when(godfatherRental.getPoints())
      .thenReturn(2);
    Rental lionKingRental =
      mock(Rental.class);
    when(lionKingRental.getPoints())
      .thenReturn(1);
    Rental scarfaceRental =
      mock(Rental.class);
    when(scarfaceRental.getPoints())
      .thenReturn(1);
    Rental notebookRental =
      mock(Rental.class);
    when(notebookRental.getPoints())
      .thenReturn(1);
    assertEquals(
      5,
      a.customer.w(
        godfatherRental, lionKingRental,
        scarfaceRental, notebookRental)
      .build().getTotalPoints());
  }
```

```java
  
  @Test
  public void getName() {
    assertEquals(
      "Jim",
      a.customer.build().getName());
  }
  
}
```
## Test Names
# Improving Test Suites
## Separating The Solitary From The Sociable
### Increasing Consistency And Speed With Solitary Unit Tests
#### Database and Filesystem Interaction
```java
public class PidWriter {
  public static void writePid(
    String filename,
    RuntimeMXBean bean) {
    try {
      writePidtoFile(filename, bean);
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
  }

  private static void writePidtoFile(
    String filename,
    RuntimeMXBean bean) throws IOException {
    FileWriter writer =
      new FileWriter(filename);
    try {
      String runtimeName = bean.getName();
      writer.write(
        runtimeName.substring(
          0, runtimeName.indexOf('@')));
    }
    finally {
      writer.close();
    }
  }
}
```
```java
public class PidWriterTest {
  @Test
  public void writePid() throws Exception {
    RuntimeMXBean bean =
      mock(RuntimeMXBean.class);
    when(bean.getName()).thenReturn("12@X");
    PidWriter.writePid(
      "/tmp/sample.pid", bean);
    assertEquals(
      "12",
      Files.readAllLines(
        Paths.get("/tmp/sample.pid"),
        Charset.defaultCharset()).get(0));
  }
}
```
#### The Solitary Unit Test
```java
public class FileWriterGateway
  extends FileWriter {

  public static boolean disallowAccess =
    false;

  public FileWriterGateway(
    String filename) throws IOException {
    super(filename);
    if (disallowAccess) {
      throw new RuntimeException(
        "access disallowed");
    }
  }
}
```
```java
public class PidWriter {
  public static void writePid(
    String filename,
    RuntimeMXBean bean) {
    try {
      writePidtoFile(filename, bean);
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
  }

  private static void writePidtoFile(
    String filename,
    RuntimeMXBean bean) throws IOException {
    
    FileWriterGateway writer =
      new FileWriterGateway(filename);
    
    try {
      String runtimeName = bean.getName();
      writer.write(
        runtimeName.substring(
          0, runtimeName.indexOf('@')));
    }
    finally {
      writer.close();
    }
  }
}
```
```java
public class PidWriterTest extends Solitary {
  
  @Test
  public void writePid() throws Exception {
    RuntimeMXBean bean =
      mock(RuntimeMXBean.class);
    when(bean.getName()).thenReturn("12@X");
    PidWriter.writePid(
      "/tmp/sample.pid", bean);
    assertEquals(
      "12",
      Files.readAllLines(
        Paths.get("/tmp/sample.pid"),
        Charset.defaultCharset()).get(0));
  }
}
```
```java
public class Solitary {
  @Before
  public void setup() {
    FileWriterGateway.disallowAccess = true;
  }
}
```
```java
public class PidWriterTest extends Solitary {
  @Test
  public void writePid() throws Exception {
    RuntimeMXBean bean =
      mock(RuntimeMXBean.class);
    when(bean.getName()).thenReturn("12@X");
    FileWriterGateway facade =
      mock(FileWriterGateway.class);
    PidWriter.writePid(facade, bean);
    verify(facade).write("12");
  }
}
```
```java
public class PidWriter {
  public static void writePid(
    String filename,
    RuntimeMXBean bean) {
    try {
      FileWriterGateway writer =
        new FileWriterGateway(filename);
      writePid(writer, bean);
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
  }

  
  public static void writePid(
    FileWriterGateway facade,
    RuntimeMXBean bean) {
    try {
      writePidtoFile(facade, bean);
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
  }
```

```java
  

  private static void writePidtoFile(
    FileWriterGateway facade,
    RuntimeMXBean bean) throws IOException {
    try {
      String runtimeName = bean.getName();
      facade.write(
        runtimeName.substring(
          0, runtimeName.indexOf('@')));
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
    finally {
      facade.close();
    }
  }
}
```
#### The Sociable Unit Test
```java
public class PidWriterTest extends Sociable {
  
  @Test
  public void writePid() throws Exception {
    RuntimeMXBean bean =
      mock(RuntimeMXBean.class);
    when(bean.getName()).thenReturn("12@X");
    PidWriter.writePid(
      "/tmp/wewut/sample.pid", bean);
    assertEquals(
      "12",
      Files.readAllLines(
        Paths.get("/tmp/wewut/sample.pid"),
        Charset.defaultCharset()).get(0));
  }
}
```
```java
public class Sociable {
  @Before
  public void setup()
    throws Exception {
    Process p;
    p = Runtime.getRuntime().exec(
      "rm -rf /tmp/wewut");
    p.waitFor();
    p = Runtime.getRuntime().exec(
      "mkdir -p /tmp/wewut");
    p.waitFor();
  }
}
```
#### Revisiting Concerns
#### Time Interaction
```java
public class Rental {

  Movie movie;
  private int daysRented;
  private boolean started;
  
  private DateTime creationDateTime;
  

  public Rental(
    Movie movie,
    int daysRented,
    
    DateTime creationDateTime) {
    
    this.movie = movie;
    this.daysRented = daysRented;
    
    this.creationDateTime = creationDateTime;
    
  }

  public Rental(Movie movie, int daysRented) {
    
    this(movie, daysRented, new DateTime());
    
  }

  
  public DateTime getCreationDateTime() {
    return creationDateTime;
  }
  
}
```
```java
public class RentalTest {
  @Test
  public void creationDateTimeNow() {
    DateTimeUtils.setCurrentMillisFixed(1000);
    Rental rental = a.rental.build();
    assertEquals(
      1000,
      rental.getCreationDateTime()
      .getMillis());
  }

  @Test
  public void creationDateTimeSet() {
    Rental rental =
      a.rental.w(
        new DateTime(199)).build();
    assertEquals(
      199,
      rental.getCreationDateTime()
      .getMillis());
  }
}
```
```java
public class RentalTest {
  @Test
  public void creationDateTimeNow() {
    DateTimeUtils.setCurrentMillisFixed(1000);
    Rental rental = a.rental.build();
    assertEquals(
      1000,
      rental.getCreationDateTime()
      .getMillis());
    
    DateTimeUtils.setCurrentMillisSystem();
    
  }

  @Test
  public void creationDateTimeSet() {
    Rental rental =
      a.rental.w(
        new DateTime(199)).build();
    assertEquals(
      199,
      rental.getCreationDateTime()
      .getMillis());
  }
}
```
```java
public class Solitary {
  @Before
  public void setup() {
    FileWriterGateway.disallowAccess = true;
    DateTimeUtils.setCurrentMillisFixed(1000);
  }
}
```
```java
public class RentalTest extends Solitary {
  @Test
  public void creationDateTimeNow() {
    Rental rental = a.rental.build();
    assertEquals(
      1000,
      rental.getCreationDateTime()
      .getMillis());
  }

  @Test
  public void creationDateTimeSet() {
    Rental rental =
      a.rental.w(
        new DateTime(199)).build();
    assertEquals(
      199,
      rental.getCreationDateTime()
      .getMillis());
  }
}
```
#### Using Speed To Your Advantage
### Avoiding Cascading Failures With Solitary Unit Tests
```java
public class CustomerTest {
  @Test
  public void noRentalsStatement() {
    assertEquals(
      "Rental record for Jim\nAmount owed " +
      "is 0.0\n" +
      "You earned 0 frequent renter points",
      a.customer.build().statement());
  }

  @Test
  public void oneRentalStatement() {
    assertEquals(
      "Rental record for Jim\n" +
      "\tGodfather 4 9.0\n" +
      "Amount owed is 9.0\n" +
      "You earned 2 frequent renter points",
      a.customer.w(
        a.rental).build().statement());
  }
```

```java
  @Test
  public void twoRentalsStatement() {
    assertEquals(
      "Rental record for Jim\n" +
      "\tGodfather 4 9.0\n" +
      "\tGodfather 4 9.0\n" +
      "Amount owed is 18.0\n" +
      "You earned 4 frequent renter points",
      a.customer.w(
        a.rental, a.rental).build()
      .statement());
  }

  @Test
  public void noRentalsGetTotalPoints() {
    assertEquals(
      0,
      a.customer.build().getTotalPoints());
  }

  @Test
  public void oneRentalGetTotalPoints() {
    assertEquals(
      2,
      a.customer.w(
        a.rental).build().getTotalPoints());
  }
```

```java
  @Test
  public void twoRentalsGetTotalPoints() {
    assertEquals(
      4,
      a.customer.w(a.rental, a.rental)
      .build()
      .getTotalPoints());
  }

  // 3 tests for htmlStatement()
  // left to the imagination
}
```
```java
public class RentalTest {
  @Test
  public void getPointsFromMovie() {
    assertEquals(
      2, a.rental.build().getPoints());
  }
}
```
```java
public class MovieTest {
  @Test
  public void getPoints() {
    assertEquals(
      2, a.movie.build().getPoints(2));
  }
}
```
```java
public class NewReleasePrice extends Price {

  @Override
  public double getCharge(int daysRented) {
    return daysRented * 3;
  }

  @Override
  public int getPoints(int daysRented) {
    if (daysRented > 1)
      
      return 3; // was 2
    
    return 1;
  }
}
```
#### Class Under Test
```java
public class CustomerTest {
  @Test
  public void noRentalsStatement() {
    assertEquals(
      "Rental record for Jim\nAmount owed " +
      "is 0.0\n" +
      "You earned 0 frequent renter points",
      a.customer.build().statement());
  }

  @Test
  public void oneRentalStatement() {
    Rental rental = mock(Rental.class);
    when(rental.getLineItem())
      .thenReturn("Godfather 4 9.0");
    when(rental.getCharge())
      .thenReturn(9.0);
    when(rental.getPoints())
      .thenReturn(2);
    assertEquals(
      "Rental record for Jim\n" +
      "\tGodfather 4 9.0\n" +
      "Amount owed is 9.0\n" +
      "You earned 2 frequent renter points",
      a.customer.w(rental).build()
      .statement());
  }
```

```java
  @Test
  public void twoRentalsStatement() {
    Rental one = mock(Rental.class);
    when(one.getLineItem())
      .thenReturn("Godfather 4 9.0");
    when(one.getCharge())
      .thenReturn(9.0);
    when(one.getPoints())
      .thenReturn(2);
    Rental two = mock(Rental.class);
    when(two.getLineItem())
      .thenReturn("Godfather 4 9.0");
    when(two.getCharge())
      .thenReturn(9.0);
    when(two.getPoints())
      .thenReturn(2);
    assertEquals(
      "Rental record for Jim\n" +
      "\tGodfather 4 9.0\n" +
      "\tGodfather 4 9.0\n" +
      "Amount owed is 18.0\n" +
      "You earned 4 frequent renter points",
      a.customer.w(one, two).build()
      .statement());
  }

  @Test
  public void noRentalsGetTotalPoints() {
    assertEquals(
      0,
      a.customer.build().getTotalPoints());
  }
```

```java
  @Test
  public void oneRentalGetTotalPoints() {
    Rental rental = mock(Rental.class);
    when(rental.getPoints())
      .thenReturn(2);
    assertEquals(
      2,
      a.customer.w(
        rental).build().getTotalPoints());
  }

  @Test
  public void twoRentalsGetTotalPoints() {
    Rental one = mock(Rental.class);
    when(one.getPoints())
      .thenReturn(2);
    Rental two = mock(Rental.class);
    when(two.getPoints())
      .thenReturn(3);
    assertEquals(
      5,
      a.customer.w(
        one, two).build().getTotalPoints());
  }
}
```
```java
public class RentalTest {
  @Test
  public void getPointsFromMovie() {
    Movie movie = mock(Movie.class);
    when(movie.getPoints(2))
      .thenReturn(2);
    assertEquals(
      2,
      a.rental.w(
        2).w(movie).build().getPoints());
  }
}
```
```java
public class Rental {

  Movie movie;
  private int daysRented;
  private boolean started;

  public Rental(
    Movie movie, int daysRented) {
    this.movie = movie;
    this.daysRented = daysRented;
  }

  public double getCharge() {
    return movie.getCharge(daysRented);
  }

  public int getPoints() {
    return
      movie.getPoints(daysRented, false);
  }

  
  public int getPoints(boolean vipFlag) {
    return
      movie.getPoints(daysRented, vipFlag);
  }
  

  public String getLineItem() {
    return
      movie.getTitle() + " " + getCharge();
  }
}
```
```java
public class CustomerTest {
  @Test
  public void noRentalsStatement() {
    assertEquals(
      "Rental record for Jim\nAmount owed " +
      "is 0.0\n" +
      "You earned 0 frequent renter points",
      a.customer.build().statement());
  }

  @Test
  public void oneRentalStatement() {
    Rental rental = mock(Rental.class);
    assertEquals(
      "Rental record for Jim\n\tnull\n" +
      "Amount owed is 0.0\n" +
      "You earned 0 frequent renter points",
      a.customer.w(
        rental).build().statement());
  }

  @Test
  public void twoRentalsStatement() {
    Rental rental = mock(Rental.class);
    assertEquals(
      "Rental record for Jim\n\tnull\n" +
      "\tnull\nAmount owed is 0.0\n" +
      "You earned 0 frequent renter points",
      a.customer.w(
        rental, rental).build().statement());
  }
```

```java
  @Test
  public void noRentalsGetTotalPoints() {
    assertEquals(
      0,
      a.customer.build().getTotalPoints());
  }

  @Test
  public void oneRentalGetTotalPoints() {
    Rental rental = mock(Rental.class);
    when(rental.getPoints())
      .thenReturn(2);
    assertEquals(
      2,
      a.customer.w(
        rental).build().getTotalPoints());
  }

  @Test
  public void twoRentalsGetTotalPoints() {
    Rental one = mock(Rental.class);
    when(one.getPoints())
      .thenReturn(2);
    Rental two = mock(Rental.class);
    when(two.getPoints())
      .thenReturn(3);
    assertEquals(
      5,
      a.customer.w(
        one, two).build().getTotalPoints());
  }
}
```
#### Revisiting the Definition of Solitary Unit Test
```java
public class MovieTest {
  @Test
  public void getChargeForChildrens() {
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(1),
      0);
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(2),
      0);
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(3),
      0);
    assertEquals(
      3.0,
      a.movie.w(
        CHILDREN).build().getCharge(4),
      0);
    assertEquals(
      4.5,
      a.movie.w(
        CHILDREN).build().getCharge(5),
      0);
  }
}
```
```java
public class Movie {

  public enum Type {
    REGULAR, NEW_RELEASE, CHILDREN, UNKNOWN;
  }

  private String title;
  Price price;

  public Movie(
    String title, Movie.Type priceCode) {
    this.title = title;
    setPriceCode(priceCode);
  }

  public String getTitle() {
    return title;
  }
```

```java
  private void setPriceCode(
    Movie.Type priceCode) {
    switch (priceCode) {
    case CHILDREN:
      price = new ChildrensPrice();
      break;
    case NEW_RELEASE:
      price = new NewReleasePrice();
      break;
    case REGULAR:
      price = new RegularPrice();
      break;
    default:
      throw new IllegalArgumentException(
        "invalid price code");
    }
  }

  public double getCharge(int daysRented) {
    return price.getCharge(daysRented);
  }

  public int getPoints(int daysRented) {
    return price.getPoints(daysRented);
  }
}
```
```java
public class ChildrensPrice extends Price {

  @Override
  public double getCharge(int daysRented) {
    double amount = 1.5;
    
    if (daysRented > 2) // *was 3*
      amount += (daysRented - 2) * 1.5;
      
    return amount;
  }
}
```
## Questionable Tests
### Testing Language Features or Standard Library Classes
```java
public class JavaTest {
  @Test
  public void arrayListGet() {
    ArrayList<Integer> list =
      new ArrayList<Integer>();
    list.add(1);
    assertEquals(
      Integer.valueOf(1), list.get(0));
  }

  @Test
  public void hashMapGet() {
    HashMap<Integer, String> map =
      new HashMap<Integer, String>();
    map.put(1, "a str");
    assertEquals("a str", map.get(1));
  }

  @Test
  public void throwCatch() {
    Exception ex = null;
    try {
      throw new RuntimeException("ex");
    } catch (Exception eCaught) {
      ex = eCaught;
    }
    assertEquals("ex", ex.getMessage());
  }
}
```
### Testing Framework Features or Classes
```java
public class JodaTest {
  @Test
  public void parseStr() {
    assertEquals(
      286347600000L,
      DateTime.parse(
        "1979-01-28").getMillis());
  }
}
```
### Testing Private Methods
## Custom Assertions
```java
public class Assert {
  public static void assertThrows(
    Class ex, Runnable runnable) {
    Exception exThrown = null;
    try {
      runnable.run();
    } catch (Exception exThrownActual) {
      exThrown = exThrownActual;
    }
    if (null == exThrown)
      fail("No exception thrown");
    else
      assertEquals(ex, exThrown.getClass());
  }
}
```
```java
public class MovieTest {
  @Test
  public void invalidTitleCustomAssertion() {
    assertThrows(
      IllegalArgumentException.class,
      () -> a.movie.w(UNKNOWN).build());
  }

  @Test
  public void invalidTitleWithoutCA() {
    Exception e = null;
    try {
      a.movie.w(UNKNOWN).build();
    } catch (Exception ex) {
      e = ex;
    }
    assertEquals(
      IllegalArgumentException.class,
      e.getClass());
  }
}
```
```java
public class AssertTest {
  @Test
  public void failIfNoThrow() {
    AssertionError e = null;
    try {
      assertThrows(
        IllegalArgumentException.class,
        mock(Runnable.class));
    } catch (AssertionError ex) {
      e = ex;
    }
    assertEquals(
      AssertionError.class,
      e.getClass());
  }

  @Test
  public void failWithMessageIfNoThrow() {
    AssertionError e = null;
    try {
      assertThrows(
        IllegalArgumentException.class,
        mock(Runnable.class));
    } catch (AssertionError ex) {
      e = ex;
    }
    assertEquals(
      "No exception thrown",
      e.getMessage());
  }
```

```java
  @Test
  public void failIfClassMismatch() {
    AssertionError e = null;
    try {
      assertThrows(
        IllegalArgumentException.class,
        () -> {
          throw new RuntimeException("");});
    } catch (AssertionError ex) {
      e = ex;
    }
    assertEquals(
      AssertionError.class,
      e.getClass());
  }
```

```java
  @Test
  public void failWithMessageIfClassWrong() {
    AssertionError e = null;
    try {
      assertThrows(
        IllegalArgumentException.class,
        () -> {
          throw new RuntimeException("");});
    } catch (AssertionError ex) {
      e = ex;
    }
    assertEquals(
      "expected:<class java.lang."+
      "IllegalArgumentException> "+
      "but was:<class java.lang."+
      "RuntimeException>",
      e.getMessage());
  }
```

```java
  @Test
  public void passWithCorrectException() {
    AssertionError e = null;
    try {
      assertThrows(
        RuntimeException.class,
        () -> {
          throw new RuntimeException("");});
    } catch (AssertionError ex) {
      e = ex;
    }
    assertEquals(null, e);
  }
}
```
### Custom Assertions on Value Objects
```java
public class MovieTest {
  @Test
  public void compareDates() {
    Movie godfather =
      a.movie.w(
        new Date(70261200000L)).build();
    assertEquals(
      "1972-03-24",
      new SimpleDateFormat(
        "yyyy-MM-dd").format(
          godfather.releaseDate()));
  }
}
```
```java
public class Assert {
  public static void assertDateWithFormat(
    String expected,
    String format,
    Date dt) {
    assertEquals(
      expected,
      new SimpleDateFormat(
        format).format(dt));
  }
}
```
```java
public class MovieTest {
  @Test
  public void compareDates() {
    Movie godfather =
      a.movie.w(
        new Date(70261200000L)).build();
    assertDateWithFormat(
      "1972-03-24",
      "yyyy-MM-dd",
      godfather.releaseDate());
  }
}
```
#### Custom Assertions for Money
```java
public class Money {
  private BigDecimal val;

  public Money(double val) {
    this(BigDecimal.valueOf(val));
  }

  public Money(BigDecimal val) {
    this.val = val;
  }

  public Money add(double d) {
    return new Money(
      val.add(BigDecimal.valueOf(d)));
  }

  public Money add(Money m) {
    return new Money(val.add(m.val));
  }

  public double toDouble() {
    return val
      .setScale(2, BigDecimal.ROUND_HALF_UP)
      .doubleValue();
  }
}
```
```java
public class MoneyTest {
  @Test
  public void doubleAddition() {
    assertEquals(
      11.0,
      a.money.w(1.0).build().add(
        10.0).toDouble(),
      0);
  }

  @Test
  public void moneyAddition() {
    assertEquals(
      11.0,
      a.money.w(1.0).build().add(
        a.money.w(10.0).build()).toDouble(),
      0);
  }

  @Test
  public void oneDecimalToDouble() {
    assertEquals(
      1.0,
      a.money.w(1.0).build().toDouble(),
      0);
  }
```

```java
  @Test
  public void twoDecimalToDouble() {
    assertEquals(
      1.12,
      a.money.w(1.12).build().toDouble(),
      0);
  }

  @Test
  public void thrDecimalUpToDouble() {
    assertEquals(
      1.12,
      a.money.w(1.123).build().toDouble(),
      0);
  }

  @Test
  public void thrDecimalDownToDouble() {
    assertEquals(
      1.13,
      a.money.w(1.125).build().toDouble(),
      0);
  }
}
```
```java
public class Movie {

  public enum Type {
    REGULAR, NEW_RELEASE, CHILDREN;
  }

  private String title;
  Price price;

  public Movie(
    String title, Movie.Type priceCode) {
    this.title = title;
    setPriceCode(priceCode);
  }

  private void setPriceCode(
    Movie.Type priceCode) {
    switch (priceCode) {
    case CHILDREN:
      price = new ChildrensPrice();
      break;
    case NEW_RELEASE:
      price = new NewReleasePrice();
      break;
    case REGULAR:
      price = new RegularPrice();
      break;
    }
  }
```

```java
  
  public Money getCharge(int daysRented) {
    return price.getCharge(daysRented);
  }
  
}
```
```java
public class MovieTest {
  @Test
  public void getChargeForChildrens1Day() {
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(
          
          1).toDouble(),
      
      0);
  }

  @Test
  public void getChargeForChildrens2Day() {
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(
          
          2).toDouble(),
      
      0);
  }

  @Test
  public void getChargeForChildrens3Day() {
    assertEquals(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(
          
          3).toDouble(),
      
      0);
  }
```

```java
  @Test
  public void getChargeForChildrens4Day() {
    assertEquals(
      3.0,
      a.movie.w(
        CHILDREN).build().getCharge(
          
          4).toDouble(),
      
      0);
  }

  @Test
  public void getChargeForChildrens5Day() {
    assertEquals(
      4.5,
      a.movie.w(
        CHILDREN).build().getCharge(
          
          5).toDouble(),
      
      0);
  }
}
```
```java
public class Assert {
  public static void assertMoney(
    double d, Money m) {
    assertEquals(d, m.toDouble(), 0);
  }
}
```
```java
public class MoneyTest {
  @Test
  public void doubleAddition() {
    assertMoney(
      11.0, a.money.w(1.0).build().add(10.0));
  }

  @Test
  public void moneyAddition() {
    assertMoney(
      11.0,
      a.money.w(1.0).build().add(
        a.money.w(10.0).build()));
  }

  @Test
  public void oneDecimalToDouble() {
    assertMoney(
      1.0, a.money.w(1.0).build());
  }

  @Test
  public void twoDecimalToDouble() {
    assertMoney(
      1.12, a.money.w(1.12).build());
  }

  @Test
  public void thrDecimalUpToDouble() {
    assertMoney(
      1.12, a.money.w(1.123).build());
  }
```

```java
  @Test
  public void thrDecimalDownToDouble() {
    assertMoney(
      1.13, a.money.w(1.125).build());
  }
}
```
```java
public class MovieTest {
  @Test
  public void getChargeForChildrens1Day() {
    assertMoney(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(1));
  }

  @Test
  public void getChargeForChildrens2Day() {
    assertMoney(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(2));
  }

  @Test
  public void getChargeForChildrens3Day() {
    assertMoney(
      1.5,
      a.movie.w(
        CHILDREN).build().getCharge(3));
  }

  @Test
  public void getChargeForChildrens4Day() {
    assertMoney(
      3.0,
      a.movie.w(
        CHILDREN).build().getCharge(4));
  }
```

```java
  @Test
  public void getChargeForChildrens5Day() {
    assertMoney(
      4.5,
      a.movie.w(
        CHILDREN).build().getCharge(5));
  }
}
```
```java
public class Rental {

  Movie movie;
  private int daysRented;

  public Rental(
    Movie movie, int daysRented) {
    this.movie = movie;
    this.daysRented = daysRented;
  }

  
  public Money getCharge() {
    return movie.getCharge(daysRented);
  }
  
}
```
```java
public class RentalTest {
  @Test
  public void getChargeFromMovie() {
    Movie movie = mock(Movie.class);
    when(movie.getCharge(any(Integer.class)))
      .thenReturn(a.money.w(1.5).build());
    assertMoney(
      1.5,
      a.rental.w(movie).build().getCharge());
  }
}
```
```java
public class Customer {

  private List<Rental> rentals =
    new ArrayList<Rental>();

  public void addRental(Rental rental) {
    rentals.add(rental);
  }

  
  public Money getTotalCharge() {
    Money total = new Money(0.0);
    for (Rental rental : rentals)
      total = total.add(rental.getCharge());
    return total;
  }
  
}
```
```java
public class CustomerTest {
  @Test
  public void chargeForNoRentals() {
    assertMoney(
      0.0,
      a.customer.build().getTotalCharge());
  }

  @Test
  public void chargeForOneRental() {
    Rental rental = mock(Rental.class);
    when(rental.getCharge())
      .thenReturn(a.money.w(2.0).build());
    assertMoney(
      2.0,
      a.customer.w(
        rental).build().getTotalCharge());
  }
```

```java
  @Test
  public void chargeForTwoRentals() {
    Rental rental1 = mock(Rental.class);
    when(rental1.getCharge())
      .thenReturn(a.money.w(2.2).build());
    Rental rental2 = mock(Rental.class);
    when(rental2.getCharge())
      .thenReturn(a.money.w(3.5).build());
    assertMoney(
      5.7,
      a.customer.w(
        rental1,
        rental2).build().getTotalCharge());
  }
}
```
## Global Definition
### Creating Domain Objects Within Tests 
#### New Is The New New
#### Object Mother
#### Test Data Builders
#### Test Data Builder Syntax
#### Test Data Builder Guidelines Revisited
```java
public class a {
  public static CustomerBuilder customer =
    new CustomerBuilder();
  public static MoneyBuilder money =
    new MoneyBuilder();

  public static class CustomerBuilder {
    Rental[] rentals;

    CustomerBuilder() {
      this(new Rental[0]);
    }

    CustomerBuilder(Rental[] rentals) {
      this.rentals = rentals;
    }

    public CustomerBuilder w(
      Rental... rentals) {
      return new CustomerBuilder(rentals);
    }

    public Customer build() {
      Customer result = new Customer();
      for (Rental rental : rentals) {
        result.addRental(rental);
      }
      return result;
    }
  }
```

```java
  public static class MoneyBuilder {
    final double val;

    MoneyBuilder() {
      this(1.0);
    }

    MoneyBuilder(double val) {
      this.val = val;
    }

    public MoneyBuilder w(double val) {
      return new MoneyBuilder(val);
    }

    public Money build() {
      return new Money(val);
    }
  }
}
```
```java
public class CustomerTest {
  @Test
  public void chargeForTwoRentals() {
    Rental rental1 = mock(Rental.class);
    when(rental1.getCharge())
      .thenReturn(a.money.w(2.2).build());
    Rental rental2 = mock(Rental.class);
    when(rental2.getCharge())
      .thenReturn(a.money.w(3.5).build());
    assertMoney(
      5.7,
      a.customer.w(
        rental1,
        rental2).build().getTotalCharge());
  }
}
```
```java
public class CustomerTest {
  @Test
  public void chargeForTwoRentals() {
    Rental rental1 = mock(Rental.class);
    when(rental1.getCharge())
      .thenReturn(a.money.w(2.2).build());
    Rental rental2 = mock(Rental.class);
    when(rental2.getCharge())
      .thenReturn(a.money.w(3.5).build());
    
    Customer customer = a.customer.build();
    customer.addRental(rental1);
    customer.addRental(rental2);
    
    assertMoney(
      5.7, customer.getTotalCharge());
  }
}
```
```java
public class Customer {
  private List<Rental> rentals =
    new ArrayList<Rental>();

  public void addRental(Rental rental) {
    rentals.add(rental);
  }

  public Money getTotalCharge() {
    Money total = new Money(0.0);
    for (Rental rental : rentals)
      total = total.add(rental.getCharge());
    return total;
  }
}
```
```java
public class Customer {
  private ArrayList<Rental> rentals =
    new ArrayList<Rental>();

  
  public Customer addRentals(
    Rental... newRentals) {
    rentals.addAll(Arrays.asList(newRentals));
    return this;
  }
  

  public Money getTotalCharge() {
    Money total = new Money(0.0);
    for (Rental rental : rentals)
      total = total.add(rental.getCharge());
    return total;
  }
}
```
```java
public class CustomerTest {
  @Test
  public void chargeForTwoRentals() {
    Rental rental1 = mock(Rental.class);
    when(rental1.getCharge())
      .thenReturn(a.money.w(2.2).build());
    Rental rental2 = mock(Rental.class);
    when(rental2.getCharge())
      .thenReturn(a.money.w(3.5).build());
    assertMoney(
      5.7,
      
      a.customer.build().addRentals(
        rental1, rental2).getTotalCharge());
    
  }
}
```
### Creating Stubs
#### Create, Stub, Return
```java
public class MockitoExtensions {
  @SuppressWarnings("unchecked")
  public static <T> T create(
    Object methodCall) {
    when(methodCall)
      .thenReturn(
        StubBuilder.current.returnValue);
    return (T)
      StubBuilder.current.mockInstance;
  }
  public static <T> StubBuilder<T> stub(
    Class<T> klass) {
    return new StubBuilder<T>(mock(klass));
  }
  public static class StubBuilder<T> {
    public static StubBuilder current;
    public final T mockInstance;
    private Object returnValue;
    public StubBuilder(T mockInstance) {
      current = this;
      this.mockInstance = mockInstance;
    }
    public T from() {
      return mockInstance;
    }
    public StubBuilder<T> returning(
      Object returnValue) {
      this.returnValue = returnValue;
      return this;
    }
  }
}
```
```java
public class CustomerTest {
  @Test
  public void chargeForTwoRentals() {
    assertMoney(
      5.7,
      a.customer.build().addRentals(
        create(
          stub(Rental.class)
          .returning(a.money.w(2.2).build())
          .from().getCharge()),
        create(
          stub(Rental.class)
          .returning(a.money.w(3.5).build())
          .from().getCharge()))
      .getTotalCharge());
  }
}
```
#### Create, Lambda, Return
```java
public class MockitoExtensions {
  public static <T> T stub(
    Class<T> klass,
    Function<T,Object> f,
    Object returnVal) {
    try {
      T result = mock(klass);
      when(f.apply(result))
        .thenReturn(returnVal);
      return result;
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }
}
```
```java
public class CustomerTest {
  @Test
  public void chargeForTwoRentals() {
    assertMoney(
      5.7,
      a.customer.build().addRentals(
        
        stub(Rental.class,
             s -> s.getCharge(),
             a.money.w(2.2).build()),
        stub(Rental.class,
             s -> s.getCharge(),
             a.money.w(3.5).build()))
      
      .getTotalCharge());
  }
}
```
### More Than Creation
# Closing Thoughts
## Broad Stack Tests 
## Test Pyramid
## Final Thoughts On ROI
## More...
# Foreword


# Preface
### Why Test?
### Who Should Read This Book
### Building on the Foundations Laid by Others
# More...

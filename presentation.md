---
author: Melker
paging: "%d / %d"
---

# Rust för Java-utvecklare

Melker Ulander

---

# Lite kort om Rust

- Version 1.0 släpptes 2015
- General purpose-språk
- Kompilerat och statiskt typat
- Lånat syntax från C och C++, och många idéer från funktionella programmeringsspråk
- Korrekthet och minnessäkerhet
- Prestanda
  - Zero cost abstractions
  - Fearless concurrency
- Används av många stora företag inklusive Amazon, Google, Meta, Discord, Dropbox, och Microsoft
  - Till exempel så utvecklas delar av Linux, Android och Windows i Rust

---

# Vad vi _inte_ kommer fokusera på

- Borrow-checker
  - Ownership
  - Lifetimes
  - `unsafe`
- Async
- Rusts otroliga ekosystem och _inbygggda_ IDE-agnostiska tooling
  - `rust-analyzer` (language server)
  - `cargo` (byggverktyg/pakethanterare)
  - `rustfmt` (autoformattering)
  - Stort utbud av crates (Rust-paket)

---

# Några problem med Java

- Hög verbosity
- `null`s och `NullPointerException`s: _billion dollar mistake_
  - `notNull`s/`noNullElements` överallt och löser ändå inte problemet
- Exceptions och `try`/`catch` är ett dålig sätt modellera väntade fel
- `final`s överallt: variabler och parametrar är muterbara by default
  - Går inte att se på en funktions signatur om den muterar sin input

---

# Rusts syntax

**Java:**

```java
private static int double(final int value) {
    final int doubled = value * 2;
    return doubled;
}
```

**Rust:**

```rust
fn double(value: i32) -> i32 {
    let doubled: i32 = value * 2;
    return doubled;
}
```

---

# Rusts syntax

## Implicit returns

Om den sista raden i ett scope inte slutar med `;` så returneras värdet.

```rust
fn double(value: i32) -> i32 {
    return value * 2;
}
```

Är samma sak som

```rust
fn double(value: i32) -> i32 {
    value * 2
}
```

---

# Primitiver i Rust

<!-- Från The Rust Programming Language https://doc.rust-lang.org/rust-by-example/primitives.html -->

- Signed integers: `i8`, `i16`, `i32`, `i64`, `i128` and `isize` (pointer size)
- Unsigned integers: `u8`, `u16`, `u32`, `u64`, `u128` and `usize` (pointer size)
- Floating point: `f32`, `f64`
- `char`: Unicode scalar values like `'a'`, `'α'` and `'∞'` (4 bytes each)
- `bool`: either `true` or `false`

---

# Referenser: `&`

En referens till ett värde föregås med `&`.

```rust
let a: bool = true;
let b: &bool = &a;
```

Tänk på `&a` som en pekare till `a`.

---

# Rusts syntax

## Immuterbarhet

Alla värden är immuterbara om vi inte explicit markerar dem `mut`:

```rust
let foo = 0;
foo += 3;
```

`cargo run` ger följande resultat:

```python
1 |     let foo = 0;
  |         ---
  |         |
  |         first assignment to `foo`
  |         help: consider making this binding mutable: `mut foo`
2 |     foo += 3;
  |     ^^^^^^^^ cannot assign twice to immutable variable
```

---

# Rusts syntax

## Immuterbarhet

Värden med `mut` framför är muterbara:

```rust
let mut foo = 0;
foo += 3;
```

---

# Rusts syntax

## Immuterbarhet

Parametrar som är muterbara måste ha `&mut` framför sig:

```rust
fn double(value: &mut i32) {
    *value *= 2;
}
```

---

# `&str` vs `String`

`&str`

- Billigare att skapa
- Statisk, eller en del av en `String`

```rust
let greeting: &str = "hello world";
```

`String`

- Dyrare att skapa
- Dynamisk

```rust
let greeting: String = "hello world".to_string();
```

---

# Structs

`struct`s är som klasser med endast klass-fält.

**Java:**

```java
class User {
    public String name;
    public String email;
    public int age;
}
```

**Rust:**

```rust
struct User {
    pub name: String,
    pub email: String,
    pub age: u8,
}
```

Vi kan ge `struct`s metoder med `impl`-block (vi kommer till detta).

---

# Structs

## Initiera en `struct`

```rust
struct User {
    pub name: String,
    pub email: String,
    pub age: u8,
}

let name = "Melker".to_string();
let email = "melker.ulander@omegapoint.se".to_string();

let user = User { // <-- här initieras den
    name,
    email,
    age: 29,
};

println!("{}", user.name); // -> Melker
```

---

# Tupler

Tupler kan innehålla 0 eller fler element.

## Tuple

```rust
let my_tuple = ()
let my_tuple = ("foo")
let my_tuple = ('c', 3, true)
```

## Struct tuple

```rust
struct Coordinate(i32, i32)

let coordinate = Coordinate(-5, 17)
```

Struct tuples är typsäkra.

---

# Enums med superkrafter

```rust
enum Direction {
    North,
    East,
    South,
    West,
}

let north = Direction::North;
```

Enums är också typsäkra.

---

# Enums med superkrafter

Enum-varianter kan vara `struct`s, tupler, eller andra `enum`s:

```rust
enum Action {
    Quit,
    Move { x: i32, y: i32 },
    Chat(String),
}

let do_quit: Action = Action::Quit;
let do_move: Action = Action::Move { x: -7, y: 42 };
let do_chat: Action = Action::Chat("hello".to_string());
```

---

# Pattern matching med `match`

```rust
enum Action {
    Quit,
    Move { x: i32, y: i32 },
    Chat(String),
}

fn perform(action: Action) {
    match action {
        Action::Quit => println!("Quit!"),
        Action::Move { x, y } => println!("Move to {x}, {y}"),
        Action::Chat(message) => println!("Chat: {message}"),
    }
}

let action = Action::Move { x: -3, y: 17 };
perform(action); // -> Move to -3, 17
```

---

# Pattern matching med `match`

`match` är "exhaustive"

```rust
enum Action {
    Quit,
    Move { x: i32, y: i32 },
    Chat(String),
    ChangeColor(u32, u32, u32), // <-- Ny variant
}


fn perform(action: Action) {
    match action { /* ... */ }
}
```

`cargo run` ger följande fel:

```python
1  error[E0004]: non-exhaustive patterns: `Action::ChangeColor(_, _, _)` not covered
 --> src/main.rs:9:11
  |
9 |     match action {
  |           ^^^^^^ pattern `Action::ChangeColor(_, _, _)` not covered
  |
...
```

---

# Pattern matching med `match`

Använd `_` för "default":

```rust
fn perform(action: Action) {
    match action {
        Action::Quit => println!("Quit!"),
        _ => {
            println!("Some other action");
        }
    }
}
```

---

# Pattern matching med `match`

Rust har riktigt pattern matching:

```rust
enum Action {
    Quit,
    Move { x: i32, y: i32 },
    Chat(String),
    ChangeColor(u32, u32, u32),
}

fn respond(action: Action) {
    let response = match action {
        Action::Move { x: 11, y: 42 } => "excellent choice",
        Action::Move { x: 72, y: 96 } => "also cool, but not as cool as (11, 42)",
        Action::Move { x: _, y: 0 } => "moving to ground level",
        Action::Move { x, y } => "guess we're moving to ({x}, {y})",
        Action::ChangeColor(123, 104, 238) => "That's my favourite color!",
        _ => "bruh",
    };

    println!("{response}")
}
```

---

# Destrukturering

```rust
// Tuples
let (a, _, c) = ('x', 3, true);
println!("a: {a}, c: {c}"); // -> a: x, c: true

// Struct tuples
let Coordinate(x, _) = Coordinate(10, 15);
println!("x: {x}"); // -> x: 10

// Structs
let user = User {
    name: "Melker".to_string(),
    email: "melker.ulander@omegapoint.se".to_string(),
    age: 29,
};

let User { name, age, .. } = user;
println!("{name} is {age} years old"); // --> Melker is 29 years old
```

---

# Metoder

I Rust så lever _data_ (`struct`s, `enum`s, etc.) och _beteende_ (metoder) i separata `impl`-block.

```rust
// Data
struct User {
    name: String,
    email: String,
    age: u8,
}

// Beteende
impl User {
    fn is_adult(&self) -> bool {
        self.age >= 18
    }
}

let user = User {
    name: "Melker".to_string(),
    email: "melker.ulander@omegapoint.se".to_string(),
    age: 29,
};
println!("{}", user.is_adult()); // -> true
```

---

# Metoder

Vi kan också ge metoder till `enum`s.

```rust
enum Shape {
    Circle(f64),
    Rectangle(f64, f64),
}

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle(radius) => std::f64::consts::PI * radius * radius,
            Shape::Rectangle(width, height) => width * height,
        }
    }
}

let circle: Shape = Shape::Circle(5.0);
let rectangle: Shape = Shape::Rectangle(3.0, 4.0);

println!("Circle area: {:.2}", circle.area()); // -> Circle area: 78.54
println!("Rectangle area: {:.2}", rectangle.area()); // -> Rectangle area: 12.00
```

---

# Traits är som interfaces med superkrafter

```rust
trait Area {
    fn area(&self) -> f64;
}

enum Shape {
    Circle(f64),
    Rectangle(f64, f64),
}

impl Area for Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle(radius) => std::f64::consts::PI * radius * radius,
            Shape::Rectangle(width, height) => width * height,
        }
    }
}

let circle: Shape = Shape::Circle(5.0);
let rectangle: Shape = Shape::Rectangle(3.0, 4.0);

println!("Circle area: {:.2}", circle.area()); // -> Circle area: 78.54
println!("Rectangle area: {:.2}", rectangle.area()); // -> Rectangle area: 12.00
```

---

# Traits är som interfaces med superkrafter

Traits låter oss också ge metoder till typer som inte är våra:

```rust
trait Area {
    fn area(&self) -> f64;
}

impl Area for &str {
    fn area(&self) -> f64 {
        self.len() as f64
    }
}

let area = "hello world".area();
println!("String area: {:.2}..?", area); // -> String area: 11.00..?
```

---

# Traits är som interfaces med superkrafter

Vi kan ge standardimplementationer till trait-metoder.

```rust
struct Point(f64, f64);

trait Area {
    fn area(&self) -> f64 {
        // Använd denna implementation om `.area()` inte implementerats
        0.0
    }
}

// Notera att vi *inte* implementerar `area()`
impl Area for Point {}

let point = Point(10.0, 15.0);
println!("Point area: {}", point.area()); // -> Point area: 0
```

---

# Traits är som interfaces med superkrafter

Traits går bara att använda om de finns i modulen, eller importeras.

---

# Polymorfism med traits

```rust
fn is_non_zero(area: &impl Area) -> bool {
    area.area() > 0.0
}
```

är syntaktiskt socker för

```rust
fn is_non_zero<A: Area>(area: &A) -> bool {
    // ...
}
```

Alternativt kan vi sätta trait bounds på egen rad med `where`:

```rust

fn is_non_zero<A>(shape: &A) -> bool
where
    A: Area + Volume,
{
    // implementation
}
```

Båda varianterna är semantiskt ekvivalenta.

---

# Polymorfism med traits

Parametrar kan också ha flera trait bounds:

```rust
fn is_non_zero<A: Area + Volume>(shape: &A) -> bool {
    // ...
}
```

---

# Polymorfism med traits

Vi kan skapa generiska `struct`s och `enum`s på samma sätt:

```rust
struct AreaWrapper<A: Area> {
    area: A,
}

enum MaybeArea<A: Area> {
    WithArea(A),
    NoArea,
}
```

---

# Ett par fantastiska inbyggda traits

## `Default`

```rust
struct Config {
    database_url: String,
    retry_count: u8,
    debug: bool,
}

impl Default for Config {
    fn default() -> Self {
        Config {
            database_url: "localhost:1234".to_string(),
            retry_count: 1,
            debug: false,
        }
    }
}

let default_config = Config::default();

let prod_config = Config {
    database_url: "production.db:5678".to_string(),
    ..Default::default()
};
```

---

# Ett par fantastiska inbyggda traits

## `Into`

```rust
trait Into<T> {
    fn into(self) -> T
}
```

---

# Ett par fantastiska inbyggda traits

## `Into`

```rust
struct Celsius(f64);
struct Fahrenheit(f64);

impl Into<Fahrenheit> for Celsius {
    fn into(self) -> Fahrenheit {
        Fahrenheit(self.0 * 1.8 + 32.0)
    }
}

fn boiling_temp_in_fahrenheit() -> Fahrenheit {
    let boiling_temp = Celsius(100.0);
    boiling_temp.into() // <- här anropas vår `.into()`
}
```

---

# Ett par fantastiska inbyggda traits

## `TryInto`

Vi kommer återkomma till `TryInto`...

---

# Makron

## Vad är makron i Rust?

- Kod som körs under kompilering och genererar ny kod
- Säkrare och kraftfullare än makron i C
  - Har tillgång till syntax-trädet (dvs inte bara text-substitution)
  - Hygieniska
- Med `cargo expand` kan vi se koden som de genereras

---

# Makron

## Funktionsliknande makron

```rust
println!("Hello {}", user.name);
```

## Attribut-liknande makron

```rust
#[...]
struct Amount(u32);
```

---

# Ett par fantastiska makron

## `#[derive(...)]`

`derive`-makrot genererar automatiska implementationer av traits.

```rust
#[derive(Eq, Ord, Clone, Debug, Hash, Serialize, Deserialize)]
struct Amount(u32);
```

```rust
let amount = Amount(42);
```

- `Eq`: `amount == amount`
- `Ord`: `amount > amount`
- `Clone`: `amount.clone()`
- `Debug`: `dbg!(amount)` för att printa som en sträng `"Amount(42)"`
- `Hash`: för hashing
- `Serialize`: serialisering
- `Deserialize`: deserialisering

<!-- Jag har utlämnat PartialEq och PartialOrd för enkelhetens skull, men de krävs för att kunna Eq and Ord -->

---

# Ett par fantastiska makron

## SQL-queries som valideras vid kompilering

```rust
struct Employee {
    first_name: String,
    last_name: String,
}

let employees = sqlx::query_as!(Employee, "SELECT first_name, last_name FROM employees")
    .fetch_all(pool)
    .await?;
```

---

# Rust har inte `null`

"I call it my billion-dollar mistake" - Tony Hoare

- Vi måste alltid anta att alla värden kan vara `null`
  - Finns inget sätt att uttrycka att en typ _aldrig_ kan vara `null`
- Ansvaret läggs på utvecklaren att göra rätt
- Problem upptäcks först vid körning (inte kompilering)
- Ett oväntat `null` kan lätt propagera - svårt att debugga

---

# Rust har inte `null`

Rust använder `Option`s istället för `null`.

```rust
pub enum Option<T> {
    None,
    Some(T),
}
```

```rust
let a: Option<i32> = Some(2);
let b: Option<i32> = None;
```

En variabel av typen `Option<i32>` kan ha värde av typen `None` eller `Some(i32)`.

En variabel av typen `i32` är **garanterad att ha ett värde**.

---

# Rust har inte `null`

```rust
fn divide(dividend: i32, divisor: i32) -> Option<i32> {
    match divisor {
        0 => None,
        _ => Some(dividend / divisor),
    }
}

let (x, y) = (3, 0);

match divide(x, y) {
    Some(result) => println!("{x} divided by {y} is {result}"),
    None => println!("Division by zero"),
};

```

---

# Rust har inte `null`

```rust
fn divide(dividend: i32, divisor: i32) -> Option<i32> {
    match divisor {
        0 => None,
        _ => Some(dividend / divisor),
    }
}

let (x, y) = (3, 0);

match divide(x, y) {
    Some(result) => println!("{x} divided by {y} is {result}"),
    None => println!("Division by zero"),
};

// Alternativt:
if let Some(result) = divide(x, y) {
    println!("{x} divided by {y} is {result}")
} else {
    println!("Division by zero"),
}
```

---

# Rust har inte exceptions

```java
// Java
final UserConfig userConfig = Config.getUserConfig();
```

- Vad kan gå fel när vi anropar `getUserConfig()`?
- I vilken `catch` hamnar vi om den kastar en exception?

---

# Rust har inte exceptions

```java
// Java
final UserConfig userConfig = Config.getUserConfig();
```

- Vad kan gå fel när vi anropar `getUserConfig()`?
- I vilken `catch` hamnar vi om den kastar en exception?

## Problemet med Exceptions och `try`/`catch`

- Exceptions döljer var fel kan (och inte kan) ske
- Vi kan inte vara säkra på att en funktion _aldrig_ kommer kasta ett fel
- `try`/`catch` gör det svårt att följa "sad path"-flödet

---

# Rust har inte exceptions

Rust använder `Result`s istället för exceptions.

```rust
enum Result<T, E> {
   Ok(T),
   Err(E),
}
```

Om en funktion returnerar ett `Result` så _måste_ vi hantera det (eller skicka vidare det).

Om en funktion _inte_ returnerar ett `Result` så är vi **garanterade att inget kan gå fel i den**.

Eftersom `Result` (och `Option`) är inbyggt så använder alla bibliotek dem.

---

# Rust har inte exceptions

## Java

```java
class UsernameTooShortException extends Exception {
  public UsernameTooShortException(String message) {
    super(message);
  }
}

class UsernameTooLongException extends Exception {
  public UsernameTooLongException(String message) {
    super(message);
  }
}
```

<!-- Java är för verbost för att koden ska få plats på en slide lol -->

---

# Rust har inte exceptions

## Java

```java
class UsernameValidator {
  public static String validateUsername(final String username) throws UsernameTooShortException, UsernameTooLongException {
    if (username.length() < 3) {
      throw new UsernameTooShortException("Username '" + username + "' is too short");
    } else if (username.length() > 15) {
      throw new UsernameTooLongException("Username '" + username + "' is too long");
    } else {
      return username.trim();
    }
  }
}
```

---

# Rust har inte exceptions

## Java

```java
public class Main {
    public static void main(String[] args) {
        String username = "  example_user ";

        try {
            String validUsername = UsernameValidator.validateUsername(username);
            System.out.println("Username '" + validUsername + "' is valid");
        } catch (UsernameTooShortException | UsernameTooLongException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

---

# Rust har inte exceptions

```rust
// Error kan ha vilken typ som helst, inklusive struct(s) eller enums
enum UsernameError {
    TooShort,
    TooLong,
}

fn validate_username(username: &str) -> Result<&str, UsernameError> {
    match username.len() {
        0..3 => Err(UsernameError::TooShort),
        16.. => Err(UsernameError::TooLong),
        _ => Ok(username.trim()),
    }
}


fn main() {
    match validate_username("  example_user ") {
        Ok(name) => println!("Username '{name}' is valid"),
        Err(UsernameError::TooShort) => println!("Username '{username}' is too short"),
        Err(UsernameError::TooLong) => println!("Username '{username}' is too long"),
    }
}
```

---

# Rusts fantastiska `?`-operator

## Java (med vavr):

```java
public static Either<String, Void> doTheThing() {
    return readInFileName()
        .flatMap(fileName -> downloadFile(fileName)
            .flatMap(file -> validateFile(file)
                .flatMap(file -> sendFile(file))
            )
        );
}
```

## Rust:

```rust
pub fn do_the_thing() -> Result<(), String> {
    let file_name = read_in_file_name()?;
    let file = download_file(&file_name)?;
    let validated_file = validate_file(&file)?;
    send_file(&validated_file)
}
```

---

# Rusts fantastiska `?`-operator

```rust
let my_value = my_result?;
```

är samma sak som

```rust
let my_value = match my_result {
    Ok(value) => value,
    Err(error) => return Err(error.into()),
};
```

---

# Rusts fantastiska `?`-operator

## `?` fungerar också på `Option`s:

```rust
let my_value = my_option?;
```

är samma sak som

```rust
let my_value = match my_option {
    Some(value) => value,
    None => return None,
};
```

---

# Rusts fantastiska `?`-operator

## `?` fungerar också på `Option`s:

```rust
struct User {
    age: Option<u16>,
}

fn birth_year(user: &User) -> Option<u16> {
    let current_year = chrono::Local::now().year() as u16;
    let birth_year = current_year - user.age?; // <-- Notera `?` här

    Some(birth_year)
}


let user1 = User { age: None };
let user2 = User { age: Some(29) };

dbg!(birth_year(&user1)); // -> None
dbg!(birth_year(&user2)); // -> Some(1995)
```

---

# `.unwrap()` låter oss packa upp `Option`s/`Result`s direkt

```rust
                        //  .parse() returnerar ett `Result`
let is_true: bool = "true".parse().unwrap();

println!("{is_true}"); // -> true
```

En `.unwrap()` av `None`/`Err(...)` kraschar programmet

Användbart när:

- Vi vet säkert att happy-path _alltid_ sker
- Eller i ett tidigt stadie när vi inte (ännu) bryr oss om sad-path

---

# `TryInto`

```rust
pub struct Username(String);

pub enum UsernameError { TooShort, TooLong }

impl TryInto<Username> for String {
    type Error = UsernameError;

    fn try_into(self) -> Result<Username, Self::Error> {
        match self.len() {
            0..3 => Err(UsernameError::TooShort),
            16.. => Err(UsernameError::TooLong),
            _ => {
                let name = self.trim().to_string();
                Ok(Username(name))
            }
        }
    }
}
```

```rust
// Det här är nu *enda sättet* att skapa ett Username - Secure By Design, någon?!
let username: Username = "melker".try_into()?;
```

---

# Closures (lambdas/anonyma funktioner)

**Java:**

```java
(num) => num * 2
```

**Rust:**

```rust
|num| num * 2
```

---

# Iteratorer

Vi vill ha alla **unika ord** i följande meningar som är exakt **4 bokstäver långa**:

```rust
let sentences = [
    "Rust is fast and memory-efficient",
    "Rust prevents segfaults",
    "Concurrency without data races",
];

let words_with_4_chars: Vec<_> = sentences
    .iter()                                           // Skapa en iterator
    .flat_map(|sentence| sentence.split_whitespace()) // Dela upp varje mening i ord
    .map(|word| word.to_lowercase())                  // Konvertera varje ord till gemener
    .filter(|word| word.len() == 4)                   // Filtrera ut ord som är längre än 4 bokstäver
    .collect::<HashSet<_>>()                          // Samla unika ord i en HashSet
    .into_iter()                                      // Konvertera tillbaka till en iterator
    .collect();                                       // Samla orden i en vektor

dbg!(words_with_4_chars); // -> [ "fast", "rust", "data" ]
```

Rust har ett **enormt** utbud av användbara iteratorer.

---

# Låt oss skriva lite kod!

## CLI som hämtar pokemon från PokeAPI

```
> mypokedex get pikachu
pikachu: 4 cm, 60 hg
```

---

# Sammanfattning

- Rust hjälper oss hitta fler buggar vid kompilering
- Kraftfullt typsystem
  - Structs
  - Enums
  - Tupler
  - Traits
  - Pattern matching
  - Typ-inferens
- `Option`s och `Result`s är säkrare sätt att representera "sad-path" på än null och exceptions
  - `?` gör dem väldigt smidiga att jobba med
- I Rust så lever data (`struct`s, `enum`s, etc.) och beteende (metoder) i separata block
- Allt är immuterbart och privat by default
- Makron är kod som kör under kompilering och kan generera kod
- Superpedagogiska felmeddelanden från `rustc`, ofta med förslag på lösning
  - Föreslår ibland t.o.m. hur din kod kan förenklas och bli mer idiomatisk
- Destrukturering

---

# Nästa steg?

- [The Book](https://doc.rust-lang.org/book/): Introduktionsbok om Rust (finns gratis som webbsida)
- [Rust By Example](https://doc.rust-lang.org/rust-by-example/): Praktiska exempel som illustrerar Rusts olika delar
- [Rustlings](https://github.com/rust-lang/rustlings/): Interaktiva övningar som lär ut Rust till nybörjare

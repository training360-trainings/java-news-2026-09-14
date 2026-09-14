class: inverse, center, middle

# Java SE verziók

## Viczián István - Training360

---

## Java 25 újdonságok

* 2026\. március

- JEP 503: Remove the 32-bit x86 Port
- JEP 506: Scoped Values
- JEP 510: Key Derivation Function API
- JEP 511: Module Import Declarations
- JEP 512: Compact Source Files and Instance Main Methods
- JEP 513: Flexible Constructor Bodies
- JEP 514: Ahead-of-Time Command-Line Ergonomics
- JEP 515: Ahead-of-Time Method Profiling
- JEP 518: JFR Cooperative Sampling
- JEP 519: Compact Object Headers
- JEP 520: JFR Method Timing & Tracing
- JEP 521: Generational Shenandoah

---

## Java 25 újdonságok - preview, experimental

- JEP 470: PEM Encodings of Cryptographic Objects (Preview)
- JEP 502: Stable Values (Preview)
- JEP 505: Structured Concurrency (Fifth Preview)
- JEP 507: Primitive Types in Patterns, instanceof, and switch (Third Preview)
- JEP 508: Vector API (Tenth Incubator)
- JEP 509: JFR CPU-Time Profiling (Experimental)

---

## Java 26 újdonságok

- JEP 500: Prepare to Make Final Mean Final
- JEP 504: Remove the Applet API
- JEP 516: Ahead-of-Time Object Caching with Any GC
- JEP 517: HTTP/3 for the HTTP Client API
- JEP 522: G1 GC: Improve Throughput by Reducing Synchronization

---

## Java 26 újdonságok - preview, experimental

- JEP 524: PEM Encodings of Cryptographic Objects (Second Preview)
- JEP 525: Structured Concurrency (Sixth Preview)
- JEP 526: Lazy Constants (Second Preview)
- JEP 529: Vector API (Eleventh Incubator)
- JEP 530: Primitive Types in Patterns, instanceof, and switch (Fourth Preview)

---

## Java 27 újdonságok

- JEP 523:	Make G1 the Default Garbage Collector in All Environments
- JEP 527:	Post-Quantum Hybrid Key Exchange for TLS 1.3
- JEP 534:	Compact Object Headers by Default
- JEP 536:	JFR In-Process Data Redaction

---

## Java 27 újdonságok - preview, experimental

- JEP 531:	Lazy Constants (Third Preview)
- JEP 532:	Primitive Types in Patterns, instanceof, and switch
(Fifth Preview)
- JEP 533:	Structured Concurrency (Seventh Preview)
- JEP 537:	Vector API (Twelfth Incubator)
- JEP 538:	PEM Encodings of Cryptographic Objects (Third Preview)

---

## JEP 506: Scoped Values

.small-code-12[
```java
@Slf4j
public class ScopedValueApplication {

    private final Random random = new Random();

    private final ScopedValue<String> requestId = ScopedValue.newInstance();

    static void main() {
        new ScopedValueApplication().run();
    }

    @SneakyThrows
    private void run() {
        try (ExecutorService executor = Executors.newFixedThreadPool(2)) {
            executor.invokeAll(IntStream.range(0, 3)
                    .mapToObj(i -> Executors.callable(this::processOrder))
                    .toList()
            );
        }
    }

    @SneakyThrows
    private void processOrder() {
        String id = UUID.randomUUID().toString();
        log.info("process: {}", id);
        Thread.sleep(random.nextInt(1000));
        ScopedValue.where(requestId, id).run(this::saveOrder);
    }

    private void saveOrder() {
        String id = requestId.get();        
        log.info("save: {}", id);
    }
}
```
]

---

## JEP 510: Key Derivation Function API

* PBKDF2 (Password-Based Key Derivation Function 2) - jelszavak hash-elésére
    * salt, iterációk száma – hányszor ismételjük a számítást (pl. 100 000+)

```java
char[] password = "hunter2".toCharArray();
byte[] salt = "somesalt".getBytes();
PBEKeySpec spec = new PBEKeySpec(password, salt, 65536, 256);

SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
SecretKey key = factory.generateSecret(spec);
```

Ennél már vannak modernebbek:

- bcrypt
- scrypt
- Argon2

---

## bcrypt

- Hash algoritmus jelszavak biztonságos tárolására, amelyet 1999-ben fejlesztettek ki a Blowfish titkosító algoritmus alapján
- salt, így ugyanaz a jelszó mindig más hash-t ad, ezzel megelőzve a rainbow table támadásokat
- Adaptív komplexitás: Bcrypt beállítható úgy, hogy lassítsa a hash-elést, ami megnehezíti a brute-force támadásokat
  - Emiatt CPU intenzív
- Technikai limit kb. 72 karakter

```
$2b$12$eIX0E6rGv2G7Q6PtDqWjxuZ5Yv7wY8s5vF3t8T3x6A0Wz6h0rQ9tC
```

Részei:

- Verzió
- Cost factor, azaz a hash számítás nehézségi szintje (2^12 iteráció)
- Salt
- Hash

---

## bcrypt

DoS-olható, védekezési lehetőségek:

- Rate limit
- Account lockout

`org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder`

---

## scrypt

- Memória-intenzív is
- Java `SCryptPasswordEncoder`

`org.springframework.security.crypto.scrypt.SCryptPasswordEncoder`

---

## Argon2

Argon2 a modern jelszó-hash algoritmusok legújabb sztenderdje

- Argon2d – főleg GPU-támadás ellen véd, CPU-intenzív, memóriát kevésbé használ.
- Argon2i – főleg side-channel támadások ellen véd, memóriát jobban használ.
- Argon2id – hibrid, egyszerre védi a brute-force és a side-channel támadások ellen. Ez a legajánlottabb a jelszó-hash-re.

Tulajdonságai

- Adaptív: beállítható a CPU idő, memóriahasználat és párhuzamos szálak száma.
- Memória-intenzív: memóriát is használ, így GPU/ASIC támadás kevésbé hatékony, nem csak CPU-ra épül.
- Salt: minden jelszóhoz egyedi salt.

---

## Argon2

```
$argon2id$v=19$m=65536,t=3,p=4$<salt>$<hash>
```

- m=65536 → memóriahasználat (KB)
- t=3 → iterációk száma
- p=4 → párhuzamos szálak száma

* `org.springframework.security.crypto.argon2.Argon2PasswordEncoder` -  Bouncy castle
* `org.springframework.security.crypto.password4j.Argon2Password4jPasswordEncoder` - Password4j

---

## JEP 511: Module Import Declarations

```java
import module java.base;

public class ModuleImportDemo {

    static void main() {
        LocalDate date = LocalDate.now();
        System.out.printf("Resolved Date: %s", date);
    }
}
```

---

## JEP 512: Compact Source Files and Instance Main Methods

```java
void main() {
    IO.println("Hello World");
}
```

---

## JEP 513: Flexible Constructor Bodies

```java
class Employee extends Person {

    String officeID;

    Employee(..., int age, String officeID) {
        if (age < 18  || age > 67)
            // Now fails fast!
            throw new IllegalArgumentException(...);
        this.officeID = officeID;   // Initialize before calling superclass constructor!
        super(..., age);
    }

}
```

---

## JEP 514 és JEP 515 - Ahead-of-Time...

* AOT
* Java 25 does not yet include an AOT compiler

---

## JEP 518: JFR Cooperative Sampling

* JFR
    * Beépített teljesítmény- és diagnosztikai eszköz (OpenJDK)
    * Nagyon alacsony overhead, akár prod-on is használható (~1%)
    * Event alapú, bináris fájlba ment

* Java Flight Recordernek képes megmondani az alkalmazás, hogy melyek a "safe sampling points"

---

## JEP 519: Compact Object Headers

* Experiments conducted as part of Project Lilliput show that many workloads have average object sizes of 256 to 512 bits (32 to 64 bytes).
* Header
    * Mark word: hash code, lock state, GC age, stb.
    * Class word: mutató a class metadata-ra
* Ebből a header 96 (compressed class pointer) - 128 bit (uncompressed)

---

## JEP 519: Compact Object Headers

* Megoldás:
    * Összevonja a két mezőt
    * A class pointert tovább tömöríti: compressed 32-ből ~22 bit, hiszen nem kell minden byte-ot címezni, objektumok igazítottan
    * Voltak szabad bitek, amiket eltávolított
    * Új lightweight locking mechanizmust használ 
* 64 bit lett
* Kevesebb memóriafoglalás (10 - 20% heap megtakarítás)
* Gyorsabb GC futás

---

## JEP 520: JFR Method Timing & Tracing

Minden metódushívást rögzít

---

## JEP 521: Generational Shenandoah

* Cél: kevesebb leállás
* Fő ötlet: Shenandoah még jobban párhuzamos legyen, mint a G1
* Kulcs trükk: Brooks pointer (forwarding pointer): objektum headerben lévő plusz adat, mely az átmozgatott objektumra mutat
* Picit nagyobb memória és CPU terhelés
* Eddig nem volt generációs

---

## JEP 500: Prepare to Make Final Mean Final

* Reflection tudja módosítani
* Warning

```java
static class Employee {
    private final String name;

    public Employee(String name) {
        this.name = name;
    }
}

static void main() throws Exception {
    var employee = new Employee("John Doe");
    Field f = Employee.class.getDeclaredField("name");
    f.setAccessible(true);
    System.out.println(employee.name);  // Prints 100

    f.set(employee, "Jack Doe");
    System.out.println(employee.name);  // Prints 200
}
```

---

## JEP 504: Remove the Applet API

---

## JEP 516: Ahead-of-Time Object Caching with Any GC

* GC független megoldásra való átállás (ZGC esetén is működjön)
* Az AOT és a GC független legyen egymástól

---

## JEP 517: HTTP/3 for the HTTP Client API

`HttpClientDemo.java`

* TCP helyett QUIC (UDP alapú) protokollra épül
* A HTTP/1.1 és HTTP/2 TCP + TLS handshake-et igényel
* Gyorsabban jön meg az első byte, főleg magas latency (mobil, távoli szerver) esetén.
    * Multi region appok
    * Real-time appoknál jól jöhet
* Külön stream-ek függetlenek, egy elveszett packet nem blokkolja a többit
    * Ez jó lehet streamek esetén, nincs globális megakadás
* QUIC-ben be van építve a TLS 1.3
* HTTP/3 mindig titkosított, TLS 1.3 kötelező
* Nem IP-hez kötött, WiFi → mobilnet, egyik cella → másik cella nem kavar be
    * Mobil klienseknél
* Hatékonyabb torlódáskezelés

---

## JEP 517: HTTP/3 for the HTTP Client API

```java
var client = HttpClient.newBuilder()
        .version(HttpClient.Version.HTTP_3)
        .build();
var request = HttpRequest.newBuilder(URI.create("https://www.jtechlog.hu/"))
        .GET().build();
var response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.statusCode());
var htmlText = response.body();
System.out.println(htmlText);
```

---

## JEP 522: G1 GC: Improve Throughput by Reducing Synchronization

* A heap fel van osztva kis fix méretű blokkokra, ez a card
    * 512 byte
* card table: melyik card clean, és melyik dirty
    * dirty: történt referencia módosítás
    * GC-nek foglalkoznia kell vele
* Alkalmazás thread módosítja referencia értékadáskor
* GC is módosítja
* Szinkronizáció

---

## JEP 522: G1 GC: Improve Throughput by Reducing Synchronization

* Trükk: két card table
    * aktív (primary) – ezt írják az alkalmazás threadek
    * másodlagos (secondary) – ezt dolgozzák fel a GC threadek
* Egyszerűsödik a write barrier
    * Injektált kódrészlet, mely minden attribútum írásakor lefut
    * Ez frissíti a card table-t
    * ~50 utasítás helyett kb. ~12 utasítás

---

## Primitive Types in Patterns, instanceof, and switch

```java
int port = . . .
switch (port) {
  case 80  -> IO.println("HTTP");
  case 443 -> IO.println("HTTPS");
}
```

---

## Primitive Types in Patterns, instanceof, and switch

```java
int score = . . .
switch (score) {
  case int s when s >= 90 -> IO.println("sehr gut");
  case int s when s >= 75 -> IO.println("gut");
  case int s when s >= 60 -> IO.println("befriedigend");
  case int s when s >= 50 -> IO.println("ausreichend");
  default                 -> IO.println("nicht bestanden");
}
```

---

## Primitive Types in Patterns, instanceof, and switch

```java
double value = . . .
switch (value) {
  case byte   b -> IO.println(value + " instanceof byte:   " + b);
  case short  s -> IO.println(value + " instanceof short:  " + s);
  case char   c -> IO.println(value + " instanceof char:   " + c);
  case int    i -> IO.println(value + " instanceof int:    " + i);
  case long   l -> IO.println(value + " instanceof long:   " + l);
  case float  f -> IO.println(value + " instanceof float:  " + f);
  case double d -> IO.println(value + " instanceof double: " + d);
}
```

---

## Lazy Constants

```java
private final LazyConstant<Validator> validator =
    LazyConstant.of(this::createValidator);

private Validator createValidator() {
  return Validation.buildDefaultValidatorFactory().getValidator();
}

public Set<ConstraintViolation<Order>> validate(Order order) {
  return validator.get().validate(order); // ⟵ Here we access the lazy constant
}
```

---

## Structured Concurrency

```java
ProductPage loadProductPage(long productId)
    throws InterruptedException, ExecutionException {
  try (var scope = StructuredTaskScope.open()) {
    var detailsTask = scope.fork(() -> catalogService.getDetails(productId));
    var priceTask   = scope.fork(() -> pricingService.getPrice(productId));
    var stockTask   = scope.fork(() -> inventoryService.getStock(productId));
    scope.join();
    return ProductPage.assemble(
        detailsTask.get(), priceTask.get(), stockTask.get());
  }
}
```

---

## PEM Encodings of Cryptographic Objects

```java
PrivateKey privateKey = PEMDecoder.of()
    .withDecryption(passphrase.toCharArray())
    .decode(encryptedPrivateKeyPemEncoded, PrivateKey.class);
```

```java
String encryptedPrivateKeyPemEncoded = PEMEncoder.of()
    .withEncryption(passphrase.toCharArray())
    .encodeToString(privateKey);
```


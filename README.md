# Nova Mask Utils

Masking sensitive data is the kind of thing every service reimplements
badly, usually with a regex that mangles the last useful digit. This
library does it once, in plain Java, with per-country rules — a Peruvian
DNI and a US SSN do not mask the same way.

No framework dependency: it is a static API you can call from a Spring
service, a Quarkus resource or a plain unit test.

## What's inside

Seven data types, each with its own strategy:

| `MaskType` | Country-aware | Strategies |
|---|---|---|
| `EMAIL` | — | `EmailMaskStrategy` |
| `PHONE` | ✓ | Peru, US, generic |
| `IDENTITY_DOCUMENT` | ✓ | Peru (DNI), US (SSN), generic |
| `CREDIT_CARD` | — | `CreditCardMaskStrategy` |
| `BANK_ACCOUNT` | ✓ | Peru, IBAN, generic |
| `PERSON_NAME` | — | `PersonNameMaskStrategy` |
| `IP_ADDRESS` | — | `IpAddressMaskStrategy` |

`CountryCode` covers `PE`, `US` and `GENERIC`.

## Install

Published to GitHub Packages, so the repository needs to be declared and
authenticated with a token that has `read:packages`.

```kotlin
repositories {
    maven {
        url = uri("https://maven.pkg.github.com/ahincho/nova-java-04-mask-utils")
        credentials {
            username = providers.gradleProperty("gpr.user").orNull ?: System.getenv("GITHUB_ACTOR")
            password = providers.gradleProperty("gpr.key").orNull ?: System.getenv("GITHUB_TOKEN")
        }
    }
}

dependencies {
    implementation("pe.edu.nova.java.libs:nova-mask-utils:0.1.0-SNAPSHOT")
}
```

## Use

Three ways in, depending on how much you know about the value.

**A single value, when you know its type:**

```java
import pe.edu.nova.java.libs.mask.utils.*;

MaskResult result = MaskEngine.mask("12345678", MaskType.IDENTITY_DOCUMENT, CountryCode.PE);
```

**A whole log line, when you do not:**

```java
String safe = MaskEngine.maskLog("payment from juan@acme.pe card 4111111111111111");
```

**A whole object, driven by annotations:**

```java
record Customer(
    @Masked(MaskType.EMAIL) String email,
    @Masked(MaskType.IDENTITY_DOCUMENT) String dni,
    @SkipMasking String publicId) {}

Customer safe = MaskEngine.maskAnnotated(customer);
```

`@MaskedClass` masks every field of a type by default; `@SkipMasking`
opts a field back out.

## Errors

Everything the engine can refuse to do is a typed exception:
`InvalidFormatException`, `UnsupportedMaskTypeException`, both under
`MaskException`.

## Requirements

Java 25.

## License

Eclipse Public License 2.0 — see [LICENSE](LICENSE).

Copyright © 2026 Angel Hincho.

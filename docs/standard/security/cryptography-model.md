---
title: ".NET cryptography model"
description: Review implementations of usual cryptographic algorithms in .NET. Learn the cryptography model of object inheritance and one-shots.
ms.date: 02/26/2021
dev_langs:
  - "csharp"
  - "vb"
helpviewer_keywords:
  - "cryptography [.NET], model"
  - "encryption [.NET], model"
ms.assetid: 12fecad4-fbab-432a-bade-2f05976a2971
---
# .NET cryptography model

.NET provides implementations of many standard cryptographic algorithms.

## Object inheritance

The .NET cryptography system implements an extensible pattern of derived class inheritance. The hierarchy is as follows:

- Algorithm type class, such as <xref:System.Security.Cryptography.SymmetricAlgorithm>,  <xref:System.Security.Cryptography.AsymmetricAlgorithm>, or <xref:System.Security.Cryptography.HashAlgorithm>. This level is abstract.

- Algorithm class that inherits from an algorithm type class; for example, <xref:System.Security.Cryptography.Aes>, <xref:System.Security.Cryptography.RSA>, or <xref:System.Security.Cryptography.ECDiffieHellman>. This level is abstract.

- Implementation of an algorithm class that inherits from an algorithm class; for example, <xref:System.Security.Cryptography.AesManaged>, <xref:System.Security.Cryptography.RC2CryptoServiceProvider>, or <xref:System.Security.Cryptography.ECDiffieHellmanCng>. This level is fully implemented.

This pattern of derived classes lets you add a new algorithm or a new implementation of an existing algorithm. For example, to create a new public-key algorithm, you would inherit from the <xref:System.Security.Cryptography.AsymmetricAlgorithm> class. To create a new implementation of a specific algorithm, you would create a non-abstract derived class of that algorithm.

This model of inheritance is not being used for new kinds of primitives going forward like <xref:System.Security.Cryptography.AesGcm> or <xref:System.Security.Cryptography.Shake128>. These algorithms are `sealed`. If you need an extensibility pattern or abstraction over these types, the implementation of the abstraction is the responsibility of the developer.

## One-shots

In .NET 5 one-shot APIs were introduced for hashing and HMAC. These one-shots are simpler to use, reduce allocations or are allocation-free, are thread safe, and use the best available implementation for the platform. The hashing and HMAC primitives expose this through a static `HashData` method on the type such as <xref:System.Security.Cryptography.SHA256.HashData%2A?displayProperty=nameWithType>. The static APIs offer no built-in extensibility mechanism. If you are implementing your own algorithms, it is recommended to also offer similar static APIs of the algorithm.

The <xref:System.Security.Cryptography.RandomNumberGenerator> class also offers static methods for creating or filling buffers with cryptographic random data. These will always use the system's CSPRNG (cryptographically secure pseudorandom number generator).

## How algorithms are implemented in .NET

As an example of the different implementations available for an algorithm, consider symmetric algorithms. The base for all symmetric algorithms is <xref:System.Security.Cryptography.SymmetricAlgorithm>, which is inherited by <xref:System.Security.Cryptography.Aes>, <xref:System.Security.Cryptography.TripleDES>, and others that are no longer recommended.

<xref:System.Security.Cryptography.Aes> is inherited by <xref:System.Security.Cryptography.AesCryptoServiceProvider>, <xref:System.Security.Cryptography.AesCng>, and <xref:System.Security.Cryptography.AesManaged>.

In .NET Framework on Windows:

* `*CryptoServiceProvider` algorithm classes, such as <xref:System.Security.Cryptography.AesCryptoServiceProvider>, are wrappers around the Windows Cryptography API (CAPI) implementation of an algorithm.
* `*Cng` algorithm classes, such as <xref:System.Security.Cryptography.ECDiffieHellmanCng>, are wrappers around the Windows Cryptography Next Generation (CNG) implementation.
* `*Managed` classes, such as <xref:System.Security.Cryptography.AesManaged>, are written entirely in managed code. `*Managed` implementations are not certified by the Federal Information Processing Standards (FIPS), and may be slower than the `*CryptoServiceProvider` and `*Cng` wrapper classes.

In .NET Core and .NET 5 and later versions, all implementation classes (`*CryptoServiceProvider`, `*Managed`, and `*Cng`) are wrappers for the operating system (OS) algorithms. If the OS algorithms are FIPS-certified, then .NET uses FIPS-certified algorithms. For more information, see [Cross-Platform Cryptography](cross-platform-cryptography.md).

In most cases, you don't need to directly reference an algorithm implementation class, such as `AesCryptoServiceProvider`. The methods and properties you typically need are on the base algorithm class, such as `Aes`. Create an instance of a default implementation class by using a factory method on the base algorithm class, and refer to the base algorithm class. For example, see the highlighted line of code in the following example:

:::code language="csharp" source="snippets/encrypting-data/csharp/aes-encrypt.cs" highlight="9":::
:::code language="vb" source="snippets/encrypting-data/vb/aes-encrypt.vb" highlight="13":::

## Choose an algorithm

You can select an algorithm for different reasons: for example, for data integrity, for data privacy, or to generate a key. Symmetric and hash algorithms are intended for protecting data for either integrity reasons (protect from change) or privacy reasons (protect from viewing). Hash algorithms are used primarily for data integrity.

Here is a list of recommended algorithms by application:

- Data privacy:
  - <xref:System.Security.Cryptography.Aes>
- Data integrity:
  - <xref:System.Security.Cryptography.HMACSHA256>
  - <xref:System.Security.Cryptography.HMACSHA512>
- Digital signature:
  - <xref:System.Security.Cryptography.ECDsa>
  - <xref:System.Security.Cryptography.RSA>
- Key exchange:
  - <xref:System.Security.Cryptography.ECDiffieHellman>
  - <xref:System.Security.Cryptography.RSA>
- Random number generation:
  - <xref:System.Security.Cryptography.RandomNumberGenerator.GetBytes%2A?displayProperty=nameWithType>
  - <xref:System.Security.Cryptography.RandomNumberGenerator.Fill%2A?displayProperty=nameWithType>
- Generating a key from a password:
  - <xref:System.Security.Cryptography.Rfc2898DeriveBytes.Pbkdf2%2A?displayProperty=nameWithType>

## See also

- [Cryptographic Services](cryptographic-services.md)
- [Cross-Platform Cryptography](cross-platform-cryptography.md)
- [ASP.NET Core Data Protection](/aspnet/core/security/data-protection/introduction)

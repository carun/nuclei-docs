### Code Requests (beta)

Nuclei enables the execution of external code on the host operating system. This feature allows security researchers, pentesters, and developers to extend the capabilities of Nuclei and perform complex actions beyond the scope of regular supported protocol-based testing.

By leveraging this capability, Nuclei can interact with the underlying operating system and execute custom scripts or commands, opening up a wide range of possibilities. It enables users to perform tasks such as system-level configurations, file operations, network interactions, and more. This level of control and flexibility empowers users to tailor their security testing workflows according to their specific requirements.

However, it's important to exercise caution while utilizing this feature, as executing external code on the host operating system carries inherent risks. It is crucial to ensure that the executed code is secure, thoroughly tested, and does not pose any unintended consequences or security risks to the target system.

#### Template Signing (beta)

Template signing via the private-public key mechanism is a crucial aspect of ensuring the integrity and authenticity of templates. This mechanism involves the use of asymmetric cryptography, specifically RSA and ECDSA algorithms, to create a secure and verifiable signature.

In this process, a template author generates a private key that remains confidential and securely stored. The corresponding public key is then shared with the template consumers. When a template is created or modified, the author signs it using their private key, generating a unique signature that is attached to the template.

Template consumers can verify the authenticity and integrity of a signed template by using the author's public key. By applying the appropriate cryptographic algorithm (RSA or ECDSA), they can validate the signature and ensure that the template has not been tampered with since it was signed. This provides a level of trust, as any modifications or unauthorized changes to the template would result in a failed verification process.

By employing the private-public key mechanism, template signing adds an additional layer of security and trust to the template ecosystem. It helps establish the identity of the template author and ensures that the templates used in various systems are genuine and have not been altered maliciously.

#### How to sign a template
First it's necessary to generate a private/public key pair. You need `openssl`. Here follows the commands for the two supported signature algorithms:
```console
# ECDSA
$ openssl ecparam -name prime256v1 -genkey -noout -out private.key
$ openssl ec -in priv-key.pem -pubout > public.key

# RSA
$ ssh-keygen -t rsa
```

Then these keys need to be exported as environment variables:
```console
# OSX + Linux
export NUCLEI_SIGNATURE_PRIVATE_KEY=path/to/private.key
export NUCLEI_SIGNATURE_PUBLIC_KEY=path/to/public.key
export NUCLEI_SIGNATURE_ALGORITHM=rsa

# Windows
$env:NUCLEI_SIGNATURE_PRIVATE_KEY='path/to/private.key'
$env:NUCLEI_SIGNATURE_PUBLIC_KEY='path/to/public.key'
$env:NUCLEI_SIGNATURE_ALGORITHM='rsa'
```

Finally a template can be signed either with the `v2/cmd/sign-templates` utility or directly with nuclei with the following commands:
```console
# with sign-templates
go run -v github.com/projectdiscovery/nuclei/v2/cmd/sign-templates -t code_template.yaml -prk path/to/private.key -puk path/to/public.key -a rsa

# with nuclei
nuclei -t code_template.yaml -sign
```

This will add a comment at the bottom of the template containing the hash signature:
```yaml
# digest: 4a0a00473045022023beecb1c4ef5b3b3a4d936a689d0fa5fea35524d23bbc12001fa0b21ca2500b02210082484d006ee0663ba1c8450ff0d10eb053308137af25cde223406c3423c4e5d1
```

#### Code

In the context of template creation, a code block is used to indicate the start of the requests for the template. This block marks the beginning of the code-related instructions.

```yaml
# Start the requests for the template right here
code:
```

To execute the code, a list of engines is specified, which are searched sequentially until a valid one is found on the system. The engine names must match the corresponding binary names on the system.

```yaml
- engine:
    - py
    - python3
```

The code to be executed can be provided either as an external file or as a code snippet directly within the template.

For an external file:

```yaml
source: protocols/code/pyfile.py
```

For a code snippet:
```yaml
source: |
      import sys
      print("hello from " + sys.stdin.read())
```

The target is passed to the template via stdin, and the output of the executed code is available for further processing in matchers and extractors. In the case of the Code protocol, the body part represents all data printed to stdout during the execution of the code.

#### Matchers / Extractor Parts

Valid `part` values supported by **Code** protocol for Matchers / Extractor are - 
    
| Value            | Description                        |
|------------------|------------------------------------|
| body             | All data printed from stdout       |


#### **Example Code Template**

The provided example demonstrates the execution of a Python script within the template. The specified engines are searched in the given order, and the code snippet is executed accordingly. Additionally, a matcher is included to check if the code's stdout contains the phrase "hello from input." (input must be passed as target with nuclei)

```yaml
id: py-code-snippet

info:
  name: py-code-snippet
  author: pdteam
  severity: info
  tags: code
  description: |
    py-code-snippet

code:
  - engine:
      - py
      - python3
    source: |
      import sys
      print("hello from " + sys.stdin.read())
    
    matchers:
      - type: word
        words:
          - "hello from input"
# digest: 4a0a00473045022023beecb1c4ef5b3b3a4d936a689d0fa5fea35524d23bbc12001fa0b21ca2500b02210082484d006ee0663ba1c8450ff0d10eb053308137af25cde223406c3423c4e5d1
```
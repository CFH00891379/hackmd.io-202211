# simpleSamlphp 啟動參數

[this page](https://hackmd.io/@nu_qcIVMToaLLQ-6gTt93g/SJx3xa06Ii)


### 啟動測試用的idp container
``` cmd
docker run --name=testsamlidp_idp \
-p 8080:8080 \
-p 8443:8443 \
-e SIMPLESAMLPHP_SP_ENTITY_ID=cfh_sp \
-e SIMPLESAMLPHP_SP_ASSERTION_CONSUMER_SERVICE=http://localhost:8088/sso/saml2/relayState \
-e SIMPLESAMLPHP_SP_SINGLE_LOGOUT_SERVICE=http://localhost/simplesaml/module.php/saml/sp/saml2-logout.php/test-sp \
-d kristophjunge/test-saml-idp
```


```cmd
docker run --name=testsamlidp_idp_myhome \
-p 8080:8080 \
-p 8444:8444 \
-e SIMPLESAMLPHP_SP_ENTITY_ID=cfh_sp \
-e SIMPLESAMLPHP_SP_ASSERTION_CONSUMER_SERVICE=http://192.168.68.103:8088/sso/saml2/relayState \
-e SIMPLESAMLPHP_SP_SINGLE_LOGOUT_SERVICE=http://localhost/simplesaml/module.php/saml/sp/saml2-logout.php/test-sp \
-d kristophjunge/test-saml-idp
```



### 新增下列設定於 config/authsources.php
```php
'test-sp' => array(
        'saml:SP',
        'entityID' => 'cfh_sp',
        'idp' => 'http://localhost:8080/simplesaml/saml2/idp/metadata.php',
    ),
```
    

### 新增下列設定於 metadata/saml20-idp-remote.php

```php

$metadata['http://localhost:8080/simplesaml/saml2/idp/metadata.php'] = array(
    'name' => array(
        'en' => 'Test IdP',
    ),
    'description' => 'Test IdP',
    'SingleSignOnService' => 'http://localhost:8080/simplesaml/saml2/idp/SSOService.php',
    'SingleLogoutService' => 'http://localhost:8080/simplesaml/saml2/idp/SingleLogoutService.php',
    'certFingerprint' => '119b9e027959cdb7c662cfd075d9e2ef384e445f',
);
```

### 加密和簽證的不同
簽章跟加密是不同的，可參考下文
[簽章和加密](https://stackoverflow.com/questions/454048/what-is-the-difference-between-encrypting-and-signing-in-asymmetric-encryption) 

### SamlRequest(SR) 只有簽章
再回頭看到產生 SamlRequest(SR)過程中有沒有對其加密的動作，只看到對SR有做簽章的動作。

在opensaml的套件之中，只看到encrpt(assertion)，這出現在接收IdP的訊息做使用;其實在理論或是實務中，SR是SP請IdP發出驗證使用者的訊號，只要確定是SP發出的就可以了。

[SamlRequest不需要加密](https://stackoverflow.com/questions/8276233/is-it-recommended-to-sign-and-encrypt-saml-and-use-ssl)

```php
'assertion.encryption' => TRUE,
        'certificate' => 'server.pem'
```

```json
"Subject": {
        "NameID": {
            "_SPNameQualifier": "cfh_sp",
            "_Format": "urn:oasis:names:tc:SAML:2.0:nameid-format:transient",
            "__prefix": "saml",
            "__text": "_e27f00709a2e034fdee87ac12ed68c763aa06b2b97"
        },
        "SubjectConfirmation": {
            "SubjectConfirmationData": {
                "_NotOnOrAfter": "2022-11-10T06:37:41Z",
                "_Recipient": "http://172.30.138.225:8087/sso/saml2/relayState",
                "_InResponseTo": "e44d58fc-315e-44c8-8c7f-f54cff629a79",
                "__prefix": "saml"
            },
            "_Method": "urn:oasis:names:tc:SAML:2.0:cm:bearer",
            "__prefix": "saml"
        },
        "__prefix": "saml"
			},
```


-----


[sp-saml20remote.php 參數設定](https://hackmd.io/@nu_qcIVMToaLLQ-6gTt93g/HyKwxEsrj)
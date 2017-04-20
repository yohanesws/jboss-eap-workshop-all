Biasanya kita menginginkan masking sensitif informasi seperti password database

Jadi solusinya adalah:

1. Buat Vault dan simpan sensitif informasi
2. Config di masing2 node

   Buat Keystore
   ```
     keytool -genseckey -alias vault -keystore vault.keystore -storetype jceks -keyalg AES -keysize 128 -storepass redhat1! -keypass redhat1! -dname "CN=jboss-vault,OU=jboss,O=jboss,L=jakarta,c=ID"
   ```

   Buat Vault dan simpan sensitif informasi

   ```
   $ cd /apps/eap/jboss-eap-6.4/bin
   $ ./vault.sh =========================================================================
   JBoss Vault
   JBOSS_HOME: /apps/eap/jboss-eap-6.4
   JAVA: /usr/lib/jvm/java-1.7.0-openjdk/bin/java
   =========================================================================
   **********************************
   **** JBoss Vault ***************
   **********************************
   Please enter a Digit:: 0: Start Interactive Session 1: Remove Interactive Session 2: Exit
   0
   Starting an interactive session
   Enter directory to store encrypted files:/apps/eap/vault
   Enter Keystore URL:/apps/eap/vault/vault.keystore
   Enter Keystore password: vault22
   Enter Keystore password again: vault22
   Values match
   Enter 8 character salt:12345678
   Enter iteration count as a number (Eg: 44):50
   Enter Keystore Alias:vault
   Initializing Vault
   Jan 22, 2016 3:04:12 PM org.picketbox.plugins.vault.PicketBoxSecurityVault init
   ITGroup, Inc PNB Xpress Page 12 Confidential JBoss EAP Installation and Configuration
    RED HAT CONSULTING

   INFO: PBOX000361: Default Security Vault Implementation Initialized and Ready Vault Configuration in configuration file: ********************************************
   ...
   </extensions>
   <vault>
   <vault-option name="KEYSTORE_URL" value="/opt/jboss/eap/vault/vault.keystore"/>
   <vault-option name="KEYSTORE_PASSWORD" value="MASK-5WNXs8oEbrs"/> <vault-option name="KEYSTORE_ALIAS" value="vault"/>
   <vault-option name="SALT" value="12345678"/>
   <vault-option name="ITERATION_COUNT" value="50"/>
   <vault-option name="ENC_FILE_DIR" value="/opt/jboss/eap/vault/"/> </vault><management> ... ********************************************
   Vault is initialized and ready for use
   Handshake with Vault complete
   Please enter a Digit:: 0: Store a secured attribute 1: Check whether a secured attribute exists 2: Remove secured attribute 3: Exit
   0
   Task: Store a secured attribute
   Please enter secured attribute value (such as password): password
   Please enter secured attribute value (such as password) again: password
   Values match
   Enter Vault Block:XpressDB
   Enter Attribute Name:password
   Secured attribute value has been stored in vault.
   Please make note of the following:
   ********************************************
   Vault Block:XpressDB
   Attribute Name:password
   Configuration should be done as follows:
   VAULT::XpressDB::password::1
   ********************************************
   Please enter a Digit:: 0: Store a secured attribute 1: Check whether a secured attribute exists 2: Remove secured attribute 3: Exit
   ```

   Config di Host atau standalone

   ```

   <vault>
   <vault-option name="KEYSTORE_URL" value="/opt/jboss/eap/vault/vault.keystore"/>
   <vault-option name="KEYSTORE_PASSWORD" value="MASK-5WNXs8oEbrs"/> <vault-option name="KEYSTORE_ALIAS" value="vault"/>
   <vault-option name="SALT" value="12345678"/>
   <vault-option name="ITERATION_COUNT" value="50"/>
   <vault-option name="ENC_FILE_DIR" value="/opt/jboss/eap/vault/"/> </vault>
   ....
   ```

   Config di profile domain.xml atau standalone
   ```
    <subsystem xmlns="urn:jboss:domain:datasources:1.2"> <datasources>
    <datasource jndi-name="java:jboss/<jndiname>" enabled="true" use-java-context="true">
    ... <security>
    <user-name>username</user-name>
    <password>${VAULT::XpressDB::password::1}</password> </security>
    </datasource>
    ... </datasources>
   ```

Sequeunce of Deployment

As Per the Release
    esignet - v1.4.1 (mosipid/esignet:1.4.1)
    Inji Certify = v0.10.2 (mosipid/inji-certify:0.10.2)
    mimoto - v0.15.2 (mosipid/mimoto:0.15.2)
    Inji Web = v0.11.1 (mosipid/inji-web:0.11.1)
    Inji Verify = v0.10.0 (mosipid/inji-verify:0.10.0)
    datashare = 1.3.0-beta.2 (mosipid/data-share-service:1.3.0-beta.2)

As per Suggestion from Engg
    esignet - v1.4.1 (mosipid/esignet:1.4.1)
    Inji Certify = v0.10.2 (mosipid/inji-certify:0.10.2)
    mimoto - v0.17.x (mosipqa/mimoto:0.17.x)
    Inji Web = v0.12.x (mosipqa/inji-web:0.12.x)
    Inji Verify Service = v0.11.x (mosipint/inji-verify-service:0.11.x)
    Inji Verify UI = v0.11.x (mosipint/inji-verify-ui:0.11.x)
    datashare = 1.3.0-beta.2 (mosipid/data-share-service:1.3.0-beta.2)


2. Configure the dns mapping for Host Injiweb, injiverify & injicertify

3. Take Backup of Required Config Maps & Secrets
4. Take Backup of Database
   PGPASSWORD="<password>" pg_dump -h <host> -p <port> -U postgres -C --clean --if-exists mosip_esignet > mosip_esignet.camdgc-perf.dump
5. Take Backup of Softhsm PIN and Keys
6. Restore the DB Backup back to Database and check the data
   1. PGPASSWORD="<password>" psql -h <host> -p <port> -U postgres -d postgres -f mosip_esignet.camdgc-perf.dump
7. Note Down All Exisitng Images Deployment & Jobs Image detauls for esignet namespace
8. ass new configmap for global 'mosip-signup-host'
9. Run ./install-all.sh
   1. Enter recaptcha admin site key when requested
   2. Enter recaptcha admin secret key when requested
10. Make sure you have nslookup in the system where you performing deployment
11. Upgrade artifactory images as per requested version from release notes
12. Update mosip-config as per requested version from release notes
13. Deploy esignet & Mock Relying party services & UI. Make sure esignet, oidc-ui, mock replying party & UI has been Deployed

For Mimoto Deployment
    1. First Onboard Mimoto Partner. incase if partner already present then manually configure existing secrets in the namespace
?     2.  mimotooidc (oidckeystore.p12) Need to Check, how this p12 file created. There is no script for that ?
    3. add mosip.injiweb.host in default global config
    4. Update mosip-config properties of mimoto-default.properties and additional json file from inji-config repository.


For Inji Certify Deployment
    1. Check the Database Script and install or update same. while entering db password enter db_user_password
    2. Depend on the Plugin, you have to rename the DB Name. Ex: if mosipid plugin means 'inji_certify' rename to 'inji_certify_mosipid'
    2. Deploy Softhsm_certify for this module 
    3. Change configmaps config-server-share --> active_profile_env to 'default,mosipid-identity'
    4. Need to check why hsm client21.zip is downloaded here ?
    5. Create New MISP Partner same like esignet (ROOT, CERTIFY_SERVICE, CERTIFY_PARTNER) from Inji Certify and Onboard into MOSIP and generate MISP License Key
    6. Create new Secret 'certify-misp-onboarder-key' --> 'mosip-certify-misp-key' in inji-certify namespace and set previous step MISP LicenseKey here. Also configure same to Config-Server. Manual only Automatic not handled
    7. Download secret mimotooidc --> oidckeystore.p12 as a Base64 Value and Convert Base64 to File oidckeystore.p12
    8. Onboard New Auth Partner for Inji Certify mosipid partner Plugin. Download public key in pem file and convert to JWT Public Key
    9. Esignet & Inji Certify should share same Redis cache. if Not Configured or different redis configured then it will throw error
    10. Set following property in esignet-default.property as false : "mosip.esignet.cache.secure.individual-id=false" .The reason is, since same cache used by esignet & Inji Certify, certify  trying search the data by Individual ID not by a TOKEN. If you enable this property then esignet store the data by TOKEN
    11. Why New Datashare-Inji Pod required ? can not we use existing one or Deploy inji datashare as main datashare and use for both MOSIP & Inji ?
    12. Change Active_profile for Config server fetch in Inji-Web Namespace to 'default,inji-default,standalone'
    13. Change Active_profile for Config server fetch in Inji-Certify Namespace to 'default,mosipid-identity'
    14. Add following parameters in ngnix.conf in inji-web-ui
            http {
            access_log /var/log/nginx/access1.log;
            error_log /var/log/nginx/error1.log;            
            ```large_client_header_buffers 4 32k;
            proxy_buffer_size   128k;
            proxy_buffers   4 256k;
            proxy_busy_buffers_size   256k;
            proxy_headers_hash_bucket_size 128;
            proxy_headers_hash_max_size 512;```

15. Add following properties mimoto-default.properties, incase if URI Length Too long
        server.tomcat.max-http-response-header-size=1000000
        server.tomcat.max-http-header-size=262144


for Inji-Verify
    1. update esignet_redirect_url, istion hosts in values.yaml in helm folder




Issues : 

1. In Inji Certify, while deploying pod, pod getting crashloopbackoff error. it not print any error in console and Events side.
   1. There are some Certify plugins missing in artifactory. because of that pod in carshloopbackoff mode. but pod not throwing any error 



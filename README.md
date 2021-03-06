# Tackle Application Inventory application

The Application Inventory application is used to manage a portfolio of applications.  
The Application Inventory will allow applications to be linked to control entities within [tackle-controls](https://github.com/konveyor/tackle-controls) and manage the relationships between applications.  
The Application Inventory is the gateway to applications assessment (via [tackle-pathfinder](https://github.com/konveyor/tackle-pathfinder)) and analysis (via [windup](https://github.com/windup/)).

## Development

### Required components

#### PostgreSQL

First start a PostreSQL container in [Podman](https://podman.io/) executing:
```Shell
$ podman run -it --rm=true --memory-swappiness=0 \
            --name postgres-application-inventory -e POSTGRES_USER=application_inventory \
            -e POSTGRES_PASSWORD=application_inventory -e POSTGRES_DB=application_inventory_db \
            -p 5432:5432 postgres:10.6
```
It works the same with Docker just replacing `podman` with `docker` in the above command.

#### Keycloak

```Shell
$ podman run -it --name keycloak --rm \
            -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin -e KEYCLOAK_IMPORT=/tmp/keycloak/quarkus-realm.json \
            -e DB_VENDOR=h2 -p 8180:8080 -p 8543:8443 -v ./src/main/resources/keycloak:/tmp/keycloak:Z \
            jboss/keycloak:12.0.2
```

### Run the application in dev mode

You can run your application in dev mode that enables live coding using:
```Shell
$ ./mvnw quarkus:dev
```

### Call endpoints in dev mode

To do calls to application's endpoint while running it in dev mode, execute the following commands:
```Shell
$ export access_token=$(\
    curl -X POST http://localhost:8180/auth/realms/quarkus/protocol/openid-connect/token \
    --user backend-service:secret \
    -H 'content-type: application/x-www-form-urlencoded' \
    -d 'username=alice&password=alice&grant_type=password' | jq --raw-output '.access_token' \
 )
$ curl -X GET 'http://localhost:8080/application-inventory/application?description=ser&sort=name' \
  -H 'Accept: application/json' -H "Authorization: Bearer "$access_token |jq .
```

## Test

### Test coverage

To get the report of test coverage of the application's code it's a matter of activating the `coverage` Maven profile during `verify` Maven goal execution like in the following command:
```shell
$ ./mvnw verify -Pjacoco
```

at the end the report will be available opening in a browser the `target/site/jacoco-ut/index.html` file.

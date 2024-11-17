Jasypt
---

## Purpose

Connection to databases and storing other credentials in a code repository is not good practice.

Storing credentials on a production system is then required to connect to production datasources, but leaves the credentials open to anyone with access to the system.

Jasypt is a java library enabling basic encryption to projects. Using Jasypt, you can store an encrypted value for the property, which can be used by the application, but administrators will not be able to use the encrypted value.

See [Jasypt.org](http://jasypt.org) ... what's with the http? Irony.

## Questions

### How do you generate the encrypted value? 

Jasypt includes Encryptor classes implementing 
`org.jasypt.encryption.StringEncryptor` which can be used to
encrypt properties for the application.

Since the application is also decrypting the value to use it,
an encryption password is required.

The same password used for encryption must be available to
the application at runtime.


### How does the application know how to decrypt the password?

Using the encryption password, encrypted value and same algorithm
as the encryption operation.

### Sometimes crypto packages are restricted exports, where can this be used?

According to the Jasypt FAQ, 

> Although Jasypt does not implement nor distribute in any of
> its forms any cryptographic algorithms, it can use them via
> the Java Cryptography Extension API and, as such, it is classified
> under ECCN code 5D002 and approved for export under License
> Exception TSU.

So this is not advice, but as long as your application doesn't bundle java, an application using Jasypt would likely fall under similar restrictions, since you'd likely be including the jasypt libraries (don't quote this, seek legal advice elsewhere).

### What's the best place to set the Encryption password?

Jasypt documentation suggests two methods for passing the Encryption password to your app at runtime.

1. If the app is a web application, require the password be
   set during startup with some fancy servlets. I didn't read
   this in too much detail because I don't want to go that route. 
2. Temporarily set an environment variable just for startup
   until Jasypt has read and decrypted the properties. 
   1. Set an environment variable with the encryption password
   2. Start the application
   3. Un-set the environment variable. This limits the time that the property could be read on the system.

#### What about in Docker?

Ideally you would avoid having the password stored in any sources,
including bash history and files on the docker host. To accomplish
this, it would likely need to be scripted and read as input
by the script.

```bash
# start_compose.sh
echo "Enter the Encryption Password"
read -s pw

# start docker with the password loaded in the environment and wait for healthy status
docker compose up -e JASYPT_ENCRYPTION_PW=$pw -d --wait
# unset the password
docker exec -e JASYPT_ENCRYPTION_PW= my-service
```

The password is read (without displaying it, `-s`) and passed to the startup command, `docker compose up -d` which starts the services as `detached` (`-d`). Adding `--wait` ensures the service is up and healthy before we continue. This gives us confidence that the password has already been loaded and used to decrypt our app configuration.

The `docker-compose.yml` file would look something like this, with a service that includes a `healthcheck` directive.

```yml
services:
  my-service:
    # name the service to un-set the encryption password later
    name=my-service
    ...
    healthcheck:
      # do something to test your app is running
      test: ["CMD", "curl" ...]
      interval: 10s
      timeout: 5s
      retries: 3
```



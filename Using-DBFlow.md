### Overview

One of the issues with existing SQL object relational mapping (ORM) libraries is that they rely on Java reflection to define database models, table schemas, and column relationships.   DBFlow is one of the few that relies strictly on annotation processing to generate Java code based on the [[SQLiteOpenHelper|Local-Databases-with-SQLiteOpenHelper]] framework that avoids the use of reflection.   This approach results in reducing run-time performance while also saving you from having to write a lot boilerplate code normally needed to define your database tables, manage their schema changes, and performing queries.

### Setup

The section below describes how to setup using DBFlow v3, which is currently in beta.  If you are upgrading from an older version of DBFlow, read this [migration guide](https://github.com/Raizlabs/DBFlow/blob/master/usage/Migration3Guide.md).  One of the major changes is the library used to generate Java code from annotation now relies on [JavaPoet](https://github.com/square/javapoet).  The generated database and table classes now use '_' instead of '$' as the separator, which may require small adjustments to your code when upgrading.

#### Gradle configuration

The first step is to add the android-apt plugin to your root `build.gradle` file:

```gradle
buildscript {
    repositories {
      // required for this library, don't use mavenCentral()
        jcenter()
    }
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
    }
}
```

Because DBFlow3 is still not officially released, you need also add https://jitpack.io to your `allprojects` -> `repositories` dependency list:

```java
allprojects {
    repositories {
        jcenter()
        maven { url "https://jitpack.io" }
    }
}
```

Next, within your `app/build.gradle, apply the `android-apt` plugin and add DBFlow to your dependency list.  We create a separate variable to store the version number to make it easier to change later:

```gradle
apply plugin: 'com.neenbedankt.android-apt'

def dbflow_version = "3.0.0-beta2"

dependencies {
    apt "com.github.Raizlabs.DBFlow:dbflow-processor:${dbflow_version}"
    compile "com.github.Raizlabs.DBFlow:dbflow-core:${dbflow_version}"
    compile "com.github.Raizlabs.DBFlow:dbflow:${dbflow_version}"

    // sql-cipher database encryption (optional)
    compile "com.github.Raizlabs.DBFlow:dbflow-sqlcipher:${dbflow_version}"
  }
```

**Note:** Make sure to remove any previous references of DBFlow if you are upgrading.  The annotation processor has significantly changed for older versions.  If you `java.lang.NoSuchMethodError: com.raizlabs.android.dbflow.annotation.Table.tableName()Ljava/lang/String;`, then you are likely to still have included the old annotation processor in your Gradle configuration.

#### Creating the database

Annotate your class with the `@Database` decorator to declare your database.  It should contain both the name to be used for creating the table, as well as the version number.  **Note**: if you decide to change the schema for any tables you create later, you will need to bump the version number.  The version number should always be incremented (and never downgraded) to avoid conflicts with older database versions.
 
```java
@Database(name = MyDatabase.NAME, version = MyDatabase.VERSION)
public class MyDatabase {

    public static final String NAME = "MyDataBase";

    public static final int VERSION = 1;
}
```

#### Defining your Tables

The Java objects that need to be declared as models need to extend from `BaseModel`.  In addition, you should annotate the class with the database name too.   Here we show how to create an `Organization` and `User` table:

```java
@Table(database = MyDatabase.class)
public class Organization extends BaseModel {

  @Column
  @PrimaryKey
  int id;

  @Column
  String name;
}
```

We can define a `ForeignKey` relation easily.  The `saveForeignKeyModel` denotes whether to update the foreign key if the entry is also updated.  In this case, we disable this functionality:

```java
@Table(database = MyDatabase.class)
public class User extends BaseModel {

  @Column
  @PrimaryKey
  int id;

  @Column
  @ForeignKey(saveForeignKeyModel = false)
  Organization organization;
}
```

#### Using with the Parceler library

If you are using DBFlow with the [Parceler|Using Parceler] library, make sure to annotate the class with the `@Parcel(analyze={}` decorator.  Otherwise, the Parceler library will try to serialize the fields that are associated with the `BaseModel` class.  To avoid this issue, specify to Parceler exactly which class in the inheritance chain should be examined (see this [disucssion](https://github.com/johncarl81/parceler/issues/73#issuecomment-167131080) for more details):

```java
@Table(database = MyDatabase.class)
@Parcel(analyze={User.class})   // add Parceler annotation here
public class User extends BaseModel {
}
```java

#### Instantiating DBFlow

Next, we need to instantiate `DBFlow` in our main application.  If you do not have an `Application` object, create one:

```java

public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        FlowManager.init(this);
    }
}
```

Modify your `AndroidManifest.xml` file to call this Application object:

```xml
<application
        android:name=".MyApplication"
```

### Troubleshooting

#### ProGuard issues

If you are using DBFlow with [ProGuard|Configuring ProGuard]], and see `Table is not registered with a Database. Did you forget the @Table annotation?`, make sure to include this line in your ProGuard configuration:

```
-keep class * extends com.raizlabs.android.dbflow.config.DatabaseHolder { *; }
```

You can go into your `app/build/intermediate/classes/com/raizlabs/android/dbflow/config` and look for the `GeneratedDatabaseHolder.class` to understand what code is generated.  You can also set a breakpoint inside this class to understand how DBFlow works under the covers.

#### Database schema

You can also download and inspect the local database using ADB:

```bash
adb pull /data/data/com.codepath.yourappname/databases/AppDatabase*.db
sqlite3 AppDatabase_xxxx_NN.db (xxxx = debug/release/unsignedRelease, NN = version code)
```

Type `.schema` to see how the table definitions are created:

```
sqlite> .schema
CREATE TABLE android_metadata (locale TEXT);
CREATE TABLE `User`(`id` INTEGER, PRIMARY KEY(`id`));
```

### References

* <https://github.com/Raizlabs/DBFlow#usage-docs>
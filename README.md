Ivy transitive dependency retrieval study for non-jars
====

The purpose of this small project is to study Ivy's ability to retreive transitive dependencies for non-jar projects.

There are 2 levels of dependencies in this study, appropriately named `level2` and `level3` (`level1` has no dependencies):

* `level1`: these have no dependencies
* `level2`: these depend on `level1`
* `level3`: these depend on `level2`


First, the `ivysettings.xml` is the same at all levels:

```
<?xml version="1.0" encoding="UTF-8"?>
<ivy-settings>
  <property name="repo.dir" value="/tmp/ivyrepo"/>
  <settings defaultResolver="main" />
  <resolvers>
    <filesystem name="main">
      <ivy pattern="${repo.dir}/[organisation]/[module]/[revision]/ivy.xml" />
      <artifact pattern="${repo.dir}/[organisation]/[module]/[revision]/[artifact]-[revision]-[type].[ext]" />
    </filesystem>
  </resolvers>
</ivy-settings>
```
In the example above, we define a repository on the `/tmp` file system.

The `level1/ivy.xml` file is:

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<ivy-module version="2.0"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:noNamespaceSchemaLocation="http://ant.apache.org/ivy/schemas/ivy.xsd">
  <info organisation="org.martin" module="level1" revision="1.0.0"/>
  <publications>
    <artifact name="level1" type="source"  ext="txt" />
    <artifact name="level1" type="bit1.8"  ext="bit" />
    <artifact name="level1" type="zac3.1"  ext="zac" />
  </publications>
</ivy-module>
```

In the example above, we define 3 publications. There is no dependencies.

The `level2/ivy.xml` contains the dependencies to the `level1` artifacts:

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<ivy-module version="2.0"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:noNamespaceSchemaLocation="http://ant.apache.org/ivy/schemas/ivy.xsd">

  <info organisation="org.martin" module="level2" revision="2.1.0"/>

  <publications>
    <artifact name="level2" type="source"  ext="txt" />
    <artifact name="level2" type="bit1.8"  ext="bit" />
    <artifact name="level2" type="zac3.1"  ext="zac" />
  </publications>

  <!-- Dependencies have to appear after publications due to a bug in the ivy parser -->
  <dependencies>
    <dependency org="org.martin" name="level1" rev="1.0.0">
      <artifact name="level1" type="source" ext="txt" />
      <artifact name="level1" type="bit1.8" ext="bit" />
      <artifact name="level1" type="zac3.1" ext="zac" />
    </dependency>
  </dependencies>
</ivy-module>
```

And finaly, the `level3/ivy.xml` contains the dependency on the `level2` artifacts:

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<ivy-module version="2.0"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:noNamespaceSchemaLocation="http://ant.apache.org/ivy/schemas/ivy.xsd">

  <info organisation="org.martin" module="level3" revision="3.2.1"/>

  <publications>
    <artifact name="level3" type="source"  ext="txt" />
    <artifact name="level3" type="bit1.8"  ext="bit" />
    <artifact name="level3" type="zac3.1"  ext="zac" />
  </publications>

  <!-- Dependencies has to appear after publications due to a bug in the ivy parser -->
  <dependencies>
    <dependency org="org.martin" name="level2" rev="2.1.0">
      <artifact name="level2" type="source" ext="txt" />
      <artifact name="level2" type="bit1.8" ext="bit" />
      <artifact name="level2" type="zac3.1" ext="zac" />
    </dependency>
  </dependencies>
</ivy-module>

```

First, publish `level1` and `level2` with these commands:


```
cd level1
./publish
cd ../level2
./publish
```

Now for the real transitive dependency test: retrieve the `level3`
dependencies. To make transitive dependencies are fetched, we first
clear the ~/.ivy2 cache, then we retrieve the artifacts:

```
rm -rf ~/.ivy2
cd ../level3
./retrieve
```

You can see that the `level3/dependencies` folder, and the `~/.ivy2` cache both
have all the dependencies.

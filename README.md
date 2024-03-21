# transformer-rules
# Tool to assist migration from junit4 to junit5

I used [eclipse transformer](https://github.com/eclipse/transformer) to replace the junit4
package names, type names and related resource names to junit5 equivalents
in the RESTEasy source files.  Manual editing was needed for the following
items:
* All files that imported org.junit.jupiter.api.Assertions had to be
  manually reviewed and edited to move the error message from the
  first argument in the Assertions stmt to the last argument.
* Junit4's @Tag(IgnoreOnCi.class) stmt needed to be changed to
  provided a string.  I kept it simple like this, @Tag("IgnoreOnCi.class")
* The junit4 stmt, @ExtendWith(Arquillian.class) had to be manually
  edit to be @ExtendWith(ArquillianExtension.class).
* The junit4 stmt @Test(expected = some.class) is not supported in
  junit5 (i.e. "expected =" is not supported).  Such stmts were
  changed to @Test.

## Steps I used to transform a single RESTEasy module
For convenience I cloned https://github.com/eclipse/transformer and built it the
main branch locally.

* Find project's referneces to junit4 dependencies and changed to junit5
~~~~
 <!-- junit4 -->
         <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
             <scope>test</scope>
         </dependency>
 <!-- junit 5 -->        
         <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
             <scope>test</scope>
         </dependency>
~~~~
* cd into a single module and created zip file of src directory.
~~~~
      cd ~/Resteasy/testsuite/integration-tests 
      zip -r target/integration-tests-sources.zip src
~~~~  
* Transformed the (import) package names.
~~~~
`java -jar /home/rsearls/projects/transformer/org.eclipse.transformer.cli/target/org.eclipse.transformer.cli-0.6.0-SNAPSHOT.jar \
  ./target/integration-tests-sources.zip \
  ./target/integration-tests-sources-junit5.zip \
  -o -q   -tr /home/rsearls/projects/transformer-rules/rules/junit-pkg-rename.properties \
  -tf /home/rsearls/projects/transformer-rules/rules/junit-master-text.properties`
~~~~
* Created zip for next transformation pass
~~~~
cp ./target/integration-tests-sources-junit5.zip \
 ./target/integration-tests-sources-junit5-2ndPass.zip
~~~~
* Transform embedded text
~~~~
java -jar /home/rsearls/projects/transformer/org.eclipse.transformer.cli/target/org.eclipse transformer.cli-0.6.0-SNAPSHOT.jar \
  ./target/integration-tests-sources-junit5.zip \
   ./target/integration-tests-sources-junit5-2ndPass.zip \
  -o -q   -td /home/rsearls/projects/transformer-rules/rules/junit-simple-text.properties
~~~~
* Overwrite transformed source files in module src dir
~~~~
unzip ./target/integration-tests-sources-junit5-2ndPass.zip
~~~~
* Performed manual editing as described above.
* Build and test


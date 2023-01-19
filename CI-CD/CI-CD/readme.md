Распишу по порядку:
1. Знакомство с SonarQube прошло успешно, скрипт проверил, ошибки исправил, скрин - ![SuccessResult](https://github.com/Atlipoka/devops_netology/blob/main/CI-CD/CI-CD/Lecture3.png)
2. Успешно загрузил артефакты в Nexus
````
<metadata modelVersion="1.1.0">
  <groupId>netology</groupId>
  <artifactId>java</artifactId>
  <versioning>
    <latest>19.0.2</latest>
    <release>19.0.2</release>
    <versions>
      <version>19.0.1</version>
      <version>19.0.2</version>
    </versions>
    <lastUpdated>20230119130905</lastUpdated>
  </versioning>
</metadata>
````
3. Скачал и настроил Maven, отредактировал pom.xml и скаал артефакты, вид pom.xml
````
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.netology.app</groupId>
  <artifactId>simple-app</artifactId>
  <version>1.0-SNAPSHOT</version>
   <repositories>
    <repository>
      <id>my-repo</id>
      <name>maven-public</name>
      <url>http://158.160.33.162:8081/repository/maven-public/</url>
    </repository>
  </repositories>
  <dependencies>
    <dependency>
      <groupId>netology</groupId>
      <artifactId>java</artifactId>
      <version>19.0.2</version>
      <classifier>distrib</classifier>
      <type>tar.gz</type>
    </dependency>
  </dependencies>
````

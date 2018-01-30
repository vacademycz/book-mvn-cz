Tipy a triky
############

Nastavení verze Java kompilátoru
********************************

Přidáme do sekce ``<plugins>`` POM souboru:

.. code-block: xml

   ...
   <build>
       <pluginManagement>
           <plugins>
               <plugin>
                   <groupId>org.apache.maven.plugins</groupId>
                   <artifactId>maven-compiler-plugin</artifactId>
                   <configuration>
                       <source>1.7</source>
                       <target>1.7</target>
                   </configuration>
               </plugin>
           </plugins>
       </pluginManagement>
   </build>
   ...

Nastavení výchozího goal nebo phase
***********************************

Voláme stále Maven se stejným cílem nebo fází? Např.

::
    $ mvn clean package

Díky nastavení defaultního cíle nebo fáze (od Maven 2) můžeme psát jen

::
    $ mvn

pro totéž. V ``pom.xml`` v ``<defaultGoal>`` elementu musíme nastavit nejčastější goal nebo fázi::

    <build>
        ...
        <defaultGoal>clean package</defaultGoal>
        ...

.. _varovani-kodovani-souboru:

Varování "Using platform encoding, build is platform dependent"
***************************************************************

Pokud není explicitně určeno v jakém kódování jsou zdrojové soubory, Maven nás varuje hláškou
podobnou této::

    [WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources,
    i.e. build is platform dependent!

Znamená to, že použije kódování aktuálního počítače, ale to nemusí být to, ve kterém byly soubory
vytvořeny a tak může na jiném PC build selhat.

Je třeba nastavit kódování zdrojových souborů :ref:`vlastnost <properties>`
``project.build.sourceEncoding``::

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </propeties>

Spuštění Java programu z Maven
******************************

Sám Maven neumí spustit Java program (resp. třídu se ``main()`` metodou). Pokud to potřebujeme
poslouží `exec-maven-plugin <http://www.mojohaus.org/exec-maven-plugin/>`_::

    mvn exec:java -Dexec.mainClass=org.virtage.maven.App

.. _ziskani-napovedy:

Získávání nápovědy
******************

Maven obsahuje příkazy (resp. goaly) pluginu help, který nám pomůže zjistit řadu důležitých
informací pro práci.

Zjištění goals určitého pluginu
===============================

Jaké goaly plugin nabízí k použití zjistíme příkazem

::

    $ mvn help:describe -Dplugin=<plugin-name>

Příklad výpisu pro plugin jar (zkráceno)::

    $ mvn help:describe -Dplugin=jar
    [INFO] org.apache.maven.plugins:maven-jar-plugin:2.4

    Name: Maven JAR Plugin
    Description: Builds a Java Archive (JAR) file from the compiled project
    classes and resources.
    Group Id: org.apache.maven.plugins
    Artifact Id: maven-jar-plugin
    Version: 2.4
    Goal Prefix: jar

    This plugin has 5 goals:

    jar:help
    Description: Display help information on maven-jar-plugin.
        Call
        mvn jar:help -Ddetail=true -Dgoal=<goal-name>
        to display parameter details.

    jar:jar
    Description: Build a JAR from the current project.

    jar:sign
    ...

    ...

Zjištění výsledného POM
=======================

Výsledný POM vzniklý dědičností POMů zjistíme pomocí

::

    $ mvn help:effective-pom

Zjištění aktivních profilů
==========================

Vypíše všechny profily aktivované manuálně i automaticky.

::

    $ mvn help:active-profiles

Závislosti
==========

Závilosti jsou často problematické. Různé verze, závilosti závilostí atd. Maven přichází na pomoc s
řadou goalů pro analýzu závilostí.

Vypíše strom (hierarchii) závislostí::

    mvn dependency:tree

Vypíše závislosti v abecedním pořadí::

    mvn dependency:resolve

Analýza závilostí, vypíše všechny nepoužitené a nedeklarované závilosti::

    mvn dependency:analyze

Debugging (ladění) Maven
************************

Maven nabízí řadu možností, co dělat při problémech.

.. note:: Zabudovanou nápovědu k pluginům, analýzu závilostí ap. najdeme na stránce
   :ref:`ziskani-napovedy`.

Full stack trace výjimek (exceptions)
=====================================

Pokud Maven plugin nebo Maven samotný skončí výjimkou, můžeme vynutit full stack trace volbou
``-e``, např.::

    mvn clean package -e

Vypisovat debug info
====================

Volbou -X nebo -debug přinutíme Maven vypisovat všechny detaily toho, co provádí. Pozor, výpis bude
velmi dlouhý!

::

    mvn <goal> -X

Debug Maven nebo pluginů
========================

Velmi pokročilý způsob ladění Maven představuje možnost krokovat provádění pomocí JPDA debuggeru
(např. z vašeho IDE jako IntelliJ IDEA). Místo příkazu ``mvn`` použijeme ``mvnDebug``. Ve výchozí
konfiguraci bude Maven čekat na připojení debuggeru na portu 8000::

    $ /usr/share/maven/bin/mvnDebug
    Preparing to Execute Maven in Debug Mode
    Listening for transport dt_socket at address: 8000
    ...

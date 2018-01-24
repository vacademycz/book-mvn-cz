Projekty
########

.. _vytvoreni-maven-projektu:

Vytvoření Maven projektu
************************

To, že v Maven je opravdu všechno plugin dokazuje, že i vytváření Maven projektů má na starosti
plugin s jménem `archetype <http://maven.apache.org/archetype/maven-archetype-plugin/>`_, resp. jeho
goal generate. Ten očekává zadání groupId a artifactId v parametrech ``-DgroupId`` a
``-DartifactId``::

    $ mvn archetype:generate -DgroupId=... -DartifactId=...

.. note:: Ve starších návodech se setkáte s archetype:create namísto archetype:generate. Goal
   archetype:create byl zavržen (deprecated) a neměli byste ho již používat.

Když neurčíme žádný archetyp, Maven založí jednoduchou Java aplikaci "Hello world" a test (ve
skutečnosti použije zabudovaný archetyp maven-archetype-quickstart). Hello world většinou není, co
chceme a proto se naučíme třetí důležitý parametr a to ``-DarchetypeArtifactId``. Např. vytvoření
jednoduché webové Java aplikace::

    $ mvn archetype:generate -DgroupId=... -DartifactId=.. -DarchetypeArtifactId=maven-archetype-webapp

Pokud nezadáme úplně všechny argumenty goalu archetype:generate spustí se
*interaktivní mód*, kdy se nás postupně Maven dotazuje na chybějící údaje. Dokonce lze napsat jen

::

    $ mvn archetype:generate

a postupně vyplňovat jednotlivé údaje. Výstup variant příkazu achetype:generate bude podobný tomuto:

::

    $ mvn archetype:generate -DgroupId=org.virtage.maven -DartifactId=defaultGoal
    [INFO] Scanning for projects...
    [INFO]
    [INFO] ------------------------------------------------------------------------
    [INFO] Building Maven Stub Project (No POM) 1
    [INFO] ------------------------------------------------------------------------
    [INFO]
    [INFO] --- maven-archetype-plugin:2.2:create (default-cli) @ standalone-pom ---
    [WARNING] This goal is deprecated. Please use mvn archetype:generate instead
    [INFO] Defaulting package to group ID: org.virtage.maven
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Old (1.x) Archetype: maven-archetype-quickstart:RELEASE
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: org.virtage.maven
    [INFO] Parameter: packageName, Value: org.virtage.maven
    [INFO] Parameter: package, Value: org.virtage.maven
    [INFO] Parameter: artifactId, Value: defaultGoal
    [INFO] Parameter: basedir, Value: /Users/libor/Ubuntu One/Trainings/courseware/Java/Maven/training-maven
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] project created from Old (1.x) Archetype in dir: /Users/libor/Ubuntu One/Trainings/courseware/Java/Maven/training-maven/defaultGoal
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 3.065s
    [INFO] Finished at: Tue Oct 22 17:11:33 CEST 2013
    [INFO] Final Memory: 11M/112M
    [INFO] ------------------------------------------------------------------------

Příkaz vytvoří složku pojmenovanou jako groupId a v ní :ref:`standardní adresářovou strukturu
<standardni-adresarova-struktura>`::

    └── defaultGoal
        ├── pom.xml
        └── src
            ├── main
            │   └── java
            │       └── org
            │           └── virtage
            │               └── maven
            │                   └── App.java
            └── test
                └── java
                    └── org
                        └── virtage
                            └── maven
                                └── AppTest.java

Archetypy
*********

Archetypy jsou v terminologii Maven šablony pro vytváření nových projektů.

Zabudované archetypy
====================

Když zkusíte interaktivní vytvoření zjisíte, že z :ref:`centrálního repozitáře Maven
<central-repository>` máte archetypů k dispozici mnohem víc. Samotný Maven obsahuje jen několik málo
archetypů. Podívejme se na některé užitečné z nich:

* ``maven-archetype-j2ee-simple`` – velmi jednoduchá vzorová J2EE aplikace
* ``maven-archetype-quickstart`` – javovský Maven project s "Hello world" jako ukázkou
* ``maven-archetype-simple`` – prázdný javovský Maven projekt
* ``maven-archetype-webapp`` – jednoduchá Java servlet aplikace (WAR)

Seznam a podrobnosti zabudovaných archetypů hledejte na `manuálu Maven
<http://maven.apache.org/archetype/maven-archetype-bundles/>`_.

Vlastní archetyp
================

Můžete rovněž definovat vlastní nový archetyp, ale to je mimo rozsah této učebnice. Zájemce odkážeme
na `manuál Maven <http://maven.apache.org/guides/mini/guide-creating-archetypes.html>`_.

.. _mavenizace:

Mavenizace
**********

Pod pojmem mavenizace (angl. podst jm. "mavenization", resp. slov. "to mavenize") myslíme proces
převodu knihovny nebo programu do podoby Maven artifactu. Tedy aby mohl být použit jako závilost v
Maven projektu.

.. important:: Nejprve se ubezpečte, že knihovna opravdu ještě není mavenizována
   `prohledáním Maven Central <http://search.maven.org/>`_. Když ji tady nenajdete, pátrejte na
   webu projektu a zkuste také vyhledávač s "<knihovna> maven" ap. Ne všechny projekty posílají své
   artefakty do Maven Central a hostují si Maven repozitář sami.
   
Způsobů jak mavenizovat JAR se nabízí více. Ukážeme si několik možných postupů.

.. tip:: Ať zvolíte jakýkoli z následujících možností, nezapomeňte umístit JAR soubor do 
   verzovacího systému, aby byl dostupný všem.

Repozitář v projektu (in-project repository)
============================================

Nejpracnější avšak profesionální postup, který nemá nevýhody zbývajících technik. Využívá toho, že
repozitář je běžná složka a může být umístěna i v kořenové složce Maven projektu.

.. rubric:: Postup vytvoření repozitáře v projektu

Vytvoříme repozitář goalem install:install-file do složky např. ``jar-repo`` nastavenou v parametru
``-DlocalRepositoryPath``. Musíme zadat i další údaje budoucího Maven artifactu (ve Windows uveďte
příkaz na jediné řádce bez ``/``)::

    $ mvn install:install-file -DlocalRepositoryPath=jar-repo -Dfile=pre-maven-project.jar \
    -DgroupId=org.virtage -DartifactId=premaven -Dpackaging=jar -Dversion=1.0

Maven pro nás vytvoří tuto strukturu::

    jar-repo/
    └── org
        └── virtage
            └── premaven
                ├── 1.0
                │   ├── premaven-1.0.jar
                │   └── premaven-1.0.pom
                └── maven-metadata-local.xml

.. warning:: Příklad funguje s maven-install-plugin od verze 2.3.1. Pokud máte problémy nahraďte
   příkaz na ``org.apache.maven.plugins:maven-install-plugin:2.3.1:install-file``. Případně na 
   aktuální verzi v době čtení (viz stránky `maven-install-plugin
   <http://maven.apache.org/plugins/maven-install-plugin/project-summary.html>`_).

Nyní do POM přidáme repozitář uvnitř projektu:

.. code-block: xml

   <repositories>
       <repository>
           <id>jar-repo</id>
           <url>file://${basedir}/jar-repo/</url>
       </repository>
   </repositories>

A samozřejmě nakonec samotnou závilost:

.. code-block: xml

   <dependency>
       <groupId>org.virtage</groupId>
       <artifactId>premaven</artifactId>
       <version>1.0</version>
       <scope>compile</scope>
   </dependency>


Instalace do lokálního repozitáře
=================================

Nejrychlejší a nejjednodušší je instalace JARu do lokálního repozitáře (``~/.m2/repository/``) s pomocí
goalu install:install-file (na Windows na jednom řádku bez ``/``)::

    $ mvn install:install-file -Dfile=<path-to-file> -DgroupId=<group-id> \
        -DartifactId=<artifact-id> -Dversion=<version> -Dpackaging=<packaging-type>

.. caution:: Tento přístup je vhodný jen pro lokální testování. Zásadní nevýhodou je, že aby
   sestavení proběhlo úspěšně i u dalšího programátora musí JAR nějak získat a pak ručně
   nainstalovat do svého lokálního repozitáře.

System scope
============

Druhou a **ještě méně doporučenihodnou cestou** je využití oboru platnosti ``system``. Obor
platnosti ``system`` umožňuje zadat libovolnou cestu na disku v elementu ``<systemPath>``, která se
připojí do kompilační classpath.

.. code-block:: xml

   <dependency>
       <groupId>org.virtage.maven</groupId>
       <artifactId>mavenizing.systemscope</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       <scope>system</scope>
       <systemPath>/home/libor/workspace/mavenizing.systemscope/some-dirty.jar</systemPath>
   </dependency>

Cesta v ``<systemPath>`` musí být vždy absolutní, tj. platná jen pro daný počítač.

Zmírněním je možnost použív cestě :ref:`zabudovanou property <zabudovane-properties>`
``${project.basedir}`` ukazující do složky s POMem:

.. code-block:: xml

   <systemPath>${project.basedir}/jars/dirty.jar</systemPath>

System scope svou podstatou naručuje základní výhodu Maven v podobě
:ref:`transitive závislosti <transitive-zavislost>`. Proto jsou také zavržené (deprecated) a je
pravděpodobné, že budou v příštích verzí Maven úplně odstraněny.

.. _multi-module:

Multi-module Maven projekt
**************************

Maven podporuje projekt složený z více samostatných podprojektů zvaných *moduly*. Je to podobné jako
:ref:`dědičnost POMů <dedicnost-pom>`. Multi-module projekt se výborně hodí, když máte velkou
aplikaci skládající se z částí jako např.

* desktop frond-end napsaný ve Swingu
* webový front-end napsaný v `servletech a JSP <https://vacademy.cz/kurz/jae2/>`_
* společnou knihovnu (JAR)
* back-end běžící na serveru
* dokumentace a nápověda projektu

Můžete pracovat samostatně na jednotlivých částek, ale když chcete odeslat zákazníkovi novou verzi
aplikace potřebujete sestavit všechny jeho části (moduly). Maven zjistí vzájemné závilosti modulů,
sestaví správné pořadí sestavení a vytvoří potřebné výstupy. Tento mechanismus se nazývá *reactor*.
Můžete si tohoto názvu všimnout při sestavení celého multi-module projektu::

    [INFO] Reactor Summary:
    [INFO]
    [INFO] parent .................................. SUCCESS [0.002s]
    [INFO] richclient .............................. SUCCESS [6.043s]
    [INFO] webclient ............................... SUCCESS [1.324s]

Top-level POM
=============

Multi-module projekt je sám Maven projekt a tudíž má pom.xml, kterému můžeme říkat top-level POM.
Ten v sekci ``<modules>`` odkazuje na jednotlivé moduly. Jméno modulu musí odpovídat složce ve které
je modul umístěn.

.. code-block: xml
   :caption: Top-level POM

   <project>
       ...
       <modules>
           <module>richclient</module>
           <module>webclient</module>
       </modules>
       ...
   </project>

Top-level projekt vytvoříme stejně jako běžný Maven projekt::

    mvn archetype:generate -DgroupId=org.virtage.maven.multimodule -DartifactId=parent
    
Založí se složka s názvem stejným jako artifactId. V ní vymažeme složku ``src/``, protože ji
nebudeme potřebovat.

V top-level POM změníme

1. hodnotu v ``<packaging>`` z ``jar`` na ``pom``
2. nastavíme jakékoli závilosti, pluginy ap. které mají být společné všem modulům - např. pro
   JUnit, změnit verzi Java kompilátoru ap.

Vytvoříme potřebný počet modulů, neboli Maven projektů uvnitř jiného Maven projektu::

    $ mvn archetype:generate -DgroupId=org.virtage.maven.multimodule -DartifactId=richclient
    $ mvn archetype:generate -DgroupId=org.virtage.maven.multimodule -DartifactId=webclient
    $ mvn archetype:generate -DgroupId=org.virtage.maven.multimodule -DartifactId=...

Maven vytvoří složky shodné s artifactId modulů::

    ├── pom.xml
    ├── richclient
    │   ├── pom.xml
    │   └── src
            └── ...
    └── webclient
        ├── pom.xml
        └── src
            └── ...

.. tip:: V případě, že nemůžou být moduly podsložky rodičovského POM můžete použít
   ``<relativePath>`` unvitř ``<parent>`` k určení relativní cesty k rodiči. Po této možnosti
   byste však měli sáhnout jen v opodstatněném případě.

Nastavíme referenci na rodičovský POM pomocí elementu ``<parent>``:

.. code-block: xml
   :caption: POM modulu

   <project>
       ....
       <parent>
           <groupId>org.virtage.maven.multimodule</groupId>
           <artifactId>parent</artifactId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       ...   

Ostatní elementy POMu zůstavají stejné.

.. note:: Dokonce je možné, aby rodič a potomek měli různé groupId, ale není to rozhodně doporučeno.

Nyní můžete provádět jakýkoli goal nebo phase na každém modulu zvlášt nebo v kořenovém projektu a
Maven vždy provede operace ve správném pořadí dle vzájemných závislostí modulů.

Packaging (typ balení)
**********************

Packaging (a odpovídající element ``<packaging>`` v POM) určují typ balení neboli jaký bude výsledek
sestavení Maven projektu pomocí ``mvn package``. Pokud není specifikován přepokládá se ``jar``.

.. code-block: xml

   <project>
       ...
       <packaging>war</packaging>
       ...
   </project>

Zabudové packaging typy jsou:

* jar - klasický Java archív
* war - Web ARchiv, `Java EE webová aplikace <https://vacademy.cz/kurz/jae/>`_
* ear - Enterprise ARchiv
* pom - typ balení kořenového Maven projektu v :ref:`multi-module <multi-module>` projektu

Typ balení je důležitý, protože určuje :ref:`jaké goals se provedou při spuštění dané phase
<phase-goal-table>`. Např. pro balení jar se na fázi package vyvolá goal jar:jar, pro balení pom je
to site:attach-descriptor.

.. _standardni-adresarova-struktura:

Standardní adresářová struktura
*******************************

Když :ref:`vytvoříte Maven projekt <vytvoreni-maven-projektu>` založí se zároveň *standardní
adresářová struktura (standard folder layout)*. Tu mají všechny Maven projekty stejnou a proto nový
programátor nemusí studovat, kam se ukládají jaké soubory zrovna ve vaší aplikaci.

* ``pom.xml``
* ``src/`` -- složka ze zdrojovými soubory (sources) a zdroji (resources)

  * ``main/``

    * ``java/`` - zdrojové kódy aplikace
    * ``config/`` - konfigurační soubory
    * ``resources/`` - zdroje (ne-Java soubory jako ikony ap.)
    * ``webapp/`` - složka pro JSP a HTML soubory Java web aplikace

  * ``test/``

    * ``java/`` - zdrojové kódy testů
    * ``resources/`` - zdroje potřebné pro testy. Zkopírují se do ``target/test-classes/``.

* ``target/`` - výstupní složka

  * ``classes/`` - zkompilované třídy programu (``.class`` soubory)
  * ``test-classes/`` - zkompilované třídy testů a testovací zdroje
  * ``site/`` -- vygenerovaný web projektu

.. note:: Váš projekt nemusí mít všechny uvedené složky nebo jich mít naopak více. Např.
   ``src/main/webapp`` najdete jen v javovské webové aplikaci. ``target/site/`` jen, když site
   nastavíte, aby se generoval.
   
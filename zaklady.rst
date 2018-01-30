Základy Maven
#############

.. _zavislost-plugin-artifact:

Závilost, plugin, artifact
**************************

Jednou z hlavních příčin úspěchu Maven je správa závilostí. Váš program si určuje jaký další software

* potřebuje jako závilosti (*dependency, závilost*)
* potřebuje pro sestavení (*plugin* nebo někdy také *MOJO*).

Váš sestavený program se může stát závilostí pro další program atp. Stejně tak program, který píšete
může být plugin pro Maven. Proto se obecně setkáváme s pojmem *artifact (artefakt)* zahrnující jak
závilosti, tak pluginy.

Pluginy bychom mohli ještě rozdělit na pluginy potřebné k sestavení projektu a pro generování
reportů (viz `manuál Maven <http://maven.apache.org/plugins/index.html>`_).

Coordinates
***********

Pro jednoznačnou identifikaci artifactů v Maven repozitáři slouží tzv. coordinates (souřadnice)

1. ve zkráceném tvaru *groupId*:*artifactId*:*version* nebo
2. v úplném *groupId*:*artifactId*:*packaging*:*classifier*:*version*.

.. table:: Příklady zkrácených coordinates

   +------------------------+-------------------+-----------------+
   |        groupId         |    artifactId     |     version     |
   +========================+===================+=================+
   | org.eclipse.jetty      | jetty-annotations | 9.0.3.v20130506 |
   +------------------------+-------------------+-----------------+
   | org.apache.maven.doxia | doxia             | 1.0-alpha-9     |
   +------------------------+-------------------+-----------------+

.. table:: Příklady úplných coordinates

   +------------------------+-------------------+-----------+--------------+-----------------+
   |        groupId         |    artifactId     | packaging |  classifier  |     version     |
   +========================+===================+===========+==============+=================+
   | org.eclipse.jetty      | jetty-annotations |           |              | 9.0.3.v20130506 |
   +------------------------+-------------------+-----------+--------------+-----------------+
   | org.apache.maven.doxia | doxia             | pom       | experimental | 1.0-alpha-9     |
   +------------------------+-------------------+-----------+--------------+-----------------+

Všimněte si, že mohou chybět vlastnosti

1. *packaging* neboli typ balení, pak se přepokládá hodnota jar
2. *classfier* pro upřesnění jinak kolidujících artifactů (bez defaultní hodnoty)

Více v `manuálu Maven <http://maven.apache.org/pom.html#Maven_Coordinates>`_.

Závilosti
*********

Přidání závilosti
=================

Závilost na běžném artifactu i na pluginu do Maven projektu přidáváme v souboru ``pom.xml`` jako
element ``<dependency>``. V ``pom.xml`` souboru vypadá definice závilosti na pluginu takto:

.. code-block:: xml
   :caption: Příklad definice závilosti

   <project xmlns="http://maven.apache.org/POM/4.0.0"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                         http://maven.apache.org/xsd/maven-4.0.0.xsd">
     ...
     <dependencies>
       <dependency>
         <groupId>junit</groupId>
         <artifactId>junit</artifactId>
         <version>4.0</version>
         <type>jar</type>
         <scope>test</scope>
         <optional>true</optional>
       </dependency>
       ...
     </dependencies>
     ...
   </project>

.. code-block:: xml
   :caption: Příklad definice pluginu

   <project xmlns="http://maven.apache.org/POM/4.0.0"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                         http://maven.apache.org/xsd/maven-4.0.0.xsd">
     <build>
       ...
       <plugins>
         <plugin>
           <groupId>org.apache.maven.plugins</groupId>
           <artifactId>maven-jar-plugin</artifactId>
           <version>2.0</version>
           <extensions>false</extensions>
           <inherited>true</inherited>
           <configuration>
             <classifier>test</classifier>
           </configuration>
           <dependencies>...</dependencies>
           <executions>...</executions>
         </plugin>
       </plugins>
     </build>
   </project>

.. _transitive-zavislost:

Transitive závilost
===================

(Užitečný český překlad nás nenapadá.) Závilosti obvykle mají další závilosti, ty zase další atd.
atd. Stejně tak i pluginy. Skvělou vlastností Maven je, že se postará o tyto "skryté" tzv.
transitive závilosti, tedy závilosti našich závilostí. Transitive závilosti musí být pochopitelně k
nalazení ve známých :ref:`repozitářích <repozitare>`.

Scopes (obor platnosti)
=======================

Možná jste si povšimli elementu ``<scope>`` ve výše uvedených příkladech závislostí. Každá závilost
platí pouze v určitném scope (oboru platnosti). Základními zabudovanými pěti typy oborů platnosti
jsou:

.. table:: Zabudované scope (obory platnosti) závilostí
   
   +----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
   | Obor platnosti |                                                                          Popis                                                                           |
   +================+==========================================================================================================================================================+
   | compile        | Nejčastější. Závilost je potřebná pro zkompilování.                                                                                                      |
   +----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
   | test           | Závilost potřebná jen pro spuštění testů.                                                                                                                |
   +----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
   | provided       | Potřebná pro kompilaci, ale nebude součástí výsledného JAR/WAR/... Závilost bude během spuštění dostupná na classpath (poskytne aplikační kontejner ap.) |
   +----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
   | system         | Potřebná pro kompilaci, ale cestu musíme zadat z místa na disku. V praxivhodná jen při :ref:`mavenizaci <mavenizace>`.                                  |
   +----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
   | import         | Speciální obor platnosti pro :ref:`multi-module projekty <multi-module>` (typ balení pom).                                                               |
   +----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+

Pořadí závislostí
=================

Pozor na to, že na pořadí závilostí záleží! Tak, jak je napíšeme v ``pom.xml``, v takovém pořadí
budou v classpath. To může být důležité např. pro knihovny upravující třídy (weaving) jako třeba při
`testování <https://vacademy.cz/kurz/jt/>`_ s knihovnou `JMockit <http://jmockit.org>`_.

.. _lifecycle-phase-plugin-goal:

Lifecycle, phase, plugin, goal
******************************

Maven rozděluje provádění sestavování, generování dokumentace a dalších operací do čtyř úrovní
hierarchie:

* **lifecycle (životní cyklus)** – nejvyšší rozdělení zobecňuje životní cyklus jakéhokoli software
  projektu bez ohledu na programovací jazyk
* **phase (fáze)** – lifecycle obsahuje kroky zvané phases
* **plugin** – jednotka balení a distribuce (typicky JAR soubor) obsahující jeden nebo více goalů
* **goal (cíl)** – konkrétní jednotlivá úloha identifikovaná ve formátu plugin:goal (např.
  ``tomcat7:run``, ``jar:sign``)

Tyto vztahy bychom tedy mohli znázornit jako

*lifecycle → phases → plugin → goals*

Všechny Maven projekty mají tři životní cykly:

1. clean (čistící)
2. default (výchozí, hlavní)
3. site (webové sídlo)

Jednotlivé fáze životních cyklu jsou pro jakýkoli Maven projekt vždy stejné. Podle typu balení
:ref:`artifactu <zavislost-plugin-artifact>` (``<packaging>`` v POM souboru) se liší napojené goals.

.. _phase-goal-table:
.. table:: Příklad připojených goalů stejné phase lišících se podle packaging typu

   +-----------+-----------+---------+----------------------------+
   | Packaging | Lifecycle |  Phase  |     Připojený goal         |
   +===========+===========+=========+============================+
   | ``jar``   | default   | package | ``jar:jar``                |
   +-----------+-----------+---------+----------------------------+
   | ``ear``   | default   | package | ``ear:ear``                |
   +-----------+-----------+---------+----------------------------+
   | ``pom``   | default   | package | ``site:attach-descriptor`` |
   +-----------+-----------+---------+----------------------------+

Více opět `manuál Mavenu <http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings>`_.

Clean lifecycle
===============

Jak napovídá název, clean lifecycle vymaže všechny vygenerované a zkompilované soubory (``.class``
ap.) z výstupní složky (``target/``).

+---------------+-------+
|     Phase     | Popis |
+===============+=======+
| ``pre-clean`` |       |
+---------------+-------+
| ``clean``     |       |
+---------------+-------+

Default lifecycle
=================

+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
|            Phase            |                                                            Popis                                                             |
+=============================+==============================================================================================================================+
| ``validate``                |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``initialize``              |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``generate-sources``        |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``process-sources``         |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``generate-resources``      |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``process-resources``       |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``compile``                 | Zkompiluje nalezené zdrojové kódy                                                                                            |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``process-classes``         |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``generate-test-sources``   |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``process-test-sources``    |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``generate-test-resources`` |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``process-test-resources``  |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``test-compile``            | Zkompiluje jednotkové (unit) testy                                                                                           |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``test``                    | .. _test-phase:                                                                                                              |
|                             |                                                                                                                              |
|                             | Spustí jednotkové (unit) testy                                                                                               |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``prepare-package``         |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``package``                 | Vytvoří distribuovatelnou podobu projektu (JAR, WAR, EAR ap.)                                                                |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``pre-integration-test``    |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``integration-test``        | Spustí integrační testy. Pokud je třeba, provede nasazení (deloy) distribuovatelné podoby projektu do testovacího prostředí. |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``post-integration-test``   |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``verify``                  |                                                                                                                              |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``install``                 | Spoustí kontroly ověřující, že distribuovatelná podoba projektu (balíček) je platná (validní)                                |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``install``                 | Umístí balíček do lokálního Maven repozitáře                                                                                 |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+
| ``deploy``                  | Nahraje balíček do vzdáleného Maven repozitáře                                                                               |
+-----------------------------+------------------------------------------------------------------------------------------------------------------------------+

Site lifecycle
==============

+-----------------+------------------------------------------+
|      Phase      |                  Popis                   |
+=================+==========================================+
| ``pre-site``    |                                          |
+-----------------+------------------------------------------+
| ``site``        | Vygeneruje informační web a HTML reporty |
+-----------------+------------------------------------------+
| ``post-site``   |                                          |
+-----------------+------------------------------------------+
| ``site-deploy`` | Nahraje vygenerovaný web na web server   |
+-----------------+------------------------------------------+

Provádění phase nebo goal
=========================

Z příkazové řádky můžeme vyvolat provádění phase nebo goalu. Více phases/goalů oddělujeme mezerou
mezi sebou.

Syntaxe pro spuštění phase je jen její název (např. ``package``), syntaxe pro goal je
*<plugin>*:*<goal>*. Např.:

  $ mvn clean jar:test-jar

provede postupně

1. všechny goals fáze clean patřící do cyklu clean
2. pouze goal test-jar z pluginu jar patřící do cyklu default

.. important:: Přímým vyvoláním goalu přeskočíme všechny fáze cyklu a provede se opravdu jen zadaný
   goal. Naopak vyvoláním celé fáze provede i všechny předchozí goals.
   
Následující příklad postupně provede všechny fáze (validate, initialize, generate-sources,
process-sources, generate-resources, process-resources) až po samotné compile::

  $ mvn compile

Soubor pom.xml
**************

Podle přítomnosti souboru ``pom.xml`` (nebo jen POM) poznáme, že projekt je Maven projekt. *POM
neboli Project Object Model* je reprezentací Maven projektu XML syntaxí. Slovo "projekt" zde má
velmi široký pojem a neznamená pouze zdrojový kód. V POMu evidujeme také řadu dalších informací o
projektu jako např.

* konfigurační soubory
* jména a role programátorů
* issue trackery
* umístění a typ verzovacího systému

Ukázka pom.xml
==============

Minimálním povinným základem každého POM je určení coordinates, tedy elementy

* ``<groupId>`` – ID skupiny, bývá nejčastěji organizace nebo organizace a projekt v Java package
  notaci (např. ``cz.virtage.utility``)
* ``<artifactId>`` – ID tohoto artifactu neboli jméno projektu (např. ``diskcleaner``)
* ``<version>`` – verze artifaktu (projektu) (např. ``1.5.2a``)

K tomu je třeba určit verzi POM modelu 4.0.0 platí pro Maven 2 a 3. Minimální platný POM by vypadal
např. takto

.. code-block: xml
   
   <project>
       <modelVersion>4.0.0</modelVersion>
       <groupId>cz.virtage.utility</groupId>
       <artifactId>diskcleaner</artifactId>
       <version>1.0-SNAPSHOT</version>
   </project>

Zkrácená ukázka pom.xml (většina elementů je nepovinných):

.. code-block: xml
   
   <project xmlns="http://maven.apache.org/POM/4.0.0"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                         http://maven.apache.org/xsd/maven-4.0.0.xsd">
     <modelVersion>4.0.0</modelVersion>
   
     <!-- The Basics -->
     <groupId>...</groupId>
     <artifactId>...</artifactId>
     <version>...</version>
     <packaging>...</packaging>
     <dependencies>...</dependencies>
     <parent>...</parent>
     <dependencyManagement>...</dependencyManagement>
     <modules>...</modules>
     <properties>...</properties>
   
     <!-- Build Settings -->
     <build>...</build>
     <reporting>...</reporting>
   
     <!-- More Project Information -->
     <name>...</name>
     <description>...</description>
     <url>...</url>
     <inceptionYear>...</inceptionYear>
     <licenses>...</licenses>
     <organization>...</organization>
     <developers>...</developers>
     <contributors>...</contributors>
   
     <!-- Environment Settings -->
     <issueManagement>...</issueManagement>
     <ciManagement>...</ciManagement>
     <mailingLists>...</mailingLists>
     <scm>...</scm>
     <prerequisites>...</prerequisites>
     <repositories>...</repositories>
     <pluginRepositories>...</pluginRepositories>
     <distributionManagement>...</distributionManagement>
     <profiles>...</profiles>
   </project>

.. _dedicnost-pom:

Dědičnost POM
=============

Pokud některé nastavení v POM projektu vynecháme, Maven slučováním postupně hledá v

* rodičovském POM (pro multi-module Maven projekty)
* "Super POM", když rodičovský chybí nebo po té co nenajde všechno nastavení v rodičovském

Super POM mj. určuje URL :ref:`The Central Repository <central-repository>`, :ref:`standardní
adresářovou strukturu <standardni-adresarova-struktura>` Maven projektu atd. Jeho obsah je
"zadrátován" ve zdrojovém kódu Mavenu a může se pro každou verzi mírně lišit.

.. tip:: Super POM vašeho Maven si můžeme prohlédnout příkazem ``mvn help:effective-pom``.

Viz `manuál Maven <http://maven.apache.org/pom.html#The_Super_POM>`_.

.. _properties:

Properties (vlastnosti)
=======================

Properties jsou velmi podobné stejné funkci v Antu nebo v operačním systému. Jsou to dvojice
klíč-hodnota, obojí typu řetězec, které nadefinujeme na jednou místě a opakovaně použijeme v POM
souboru. Hodí se pro uložení takových hodnot jako cesty, URL, uživatelská jméno ap.

Všechny vlastnosti se používají zápisem ``${_property_}`` a vytvářejí se v elementu
``<properties>``:

Definice:

.. code-block:: xml

   <properties>
       <color>green</color>
   </properties>

Použití:

.. code-block:: xml

   <build>
       <finalName>${artifactId}-${version}-${color}</finalName>
   </build>


.. _zabudovane-properties:
.. table:: Některé užitečné zabudované properties

   +------------------------------------+---------------------------------------------------------------------------------------------------------------------------+
   |             Vlastnosti             |                                                           Popis                                                           |
   +====================================+===========================================================================================================================+
   | ``${project.basedir}``             | Složka s POM souborem.                                                                                                    |
   +------------------------------------+---------------------------------------------------------------------------------------------------------------------------+
   | ``${project.artifactId}``          | Hodnota z ``<artifactId>``                                                                                                |
   +------------------------------------+---------------------------------------------------------------------------------------------------------------------------+
   | ``${project.groupId}``             | Hodnota z ``<groupId>``                                                                                                   |
   +------------------------------------+---------------------------------------------------------------------------------------------------------------------------+
   | ``${project.version}``             | Hodnota z ``<version>``                                                                                                   |
   +------------------------------------+---------------------------------------------------------------------------------------------------------------------------+
   | ``${project.build.directory}``     | Výstupní složka (obvykle ``target/``).                                                                                    |
   +------------------------------------+---------------------------------------------------------------------------------------------------------------------------+
   | ``${project.build.sourceEncoding}``| Kódování zdrojových souborů (dnes obvykle UTF-8). Maven :ref:`varuje <varovani-kodovani-souboru>`, pokud není definováno. |
   +------------------------------------+---------------------------------------------------------------------------------------------------------------------------+
   | ``${env._<proměnná>_}``            | Vyhodnotí proměnnou prostředí OS, např. ``${env.HOME}``
vrátí domovskou složku uživatele.                                 |
   +------------------------------------+---------------------------------------------------------------------------------------------------------------------------+

Např. velmi často potřebnou proměnnou pro správné fungování kompilátoru je
:ref:`project.build.sourceEncoding <varovani-kodovani-souboru>`, tedy kódování zdrojových ``.java``
souborů.

.. _repozitare:

Repozitáře
**********

Repozitář je složka na lokálním nebo vzdáleném souborovém systému, kterou Maven udržuje a používá k
vyhledávání a ukládání závilostí a pluginů. Obsahem složky repozitáře je mnoho dalších složek a
souborů se speciálním významem pro Maven.

Lokální repozitář
=================

Vždy přítomný lokální repozitář je v ``~/.m2/repository/``, který funguje v zásadě jako cache a
najdete zde dříve použité stažené artefakty ze vzdálených repozitářů.

.. _central-repository:

The Central Repository
======================

Druhý repozitář, který nemusíme nastavovat (definuje ho :ref:`Super POM <dedicnost-pom>`) a přesto v
něm Maven bude hledat se nazývá The Central Repository

* URL pro člověka (webový prohlížeč): http://search.maven.org/
* URL pro Maven: http://repo.maven.apache.org/maven2

V něm jsou prakticky všechny myslitelné knihovny a závilosti, které můžeme potřebovat.

Občas přesto narazíme na to, že nějaký software v The Central Repository chybí (buď třetí strany,
proprietární nebo náš vlastní software) a je třeba ho tzv. :ref:`mavenizovat <mavenizace>`.

Vlastní repozitář
=================

Je možné vytvářet vlastní repozitáře, např. v rámci firmy. Lze to sice jen z příkazové řádky, ale to
se hodí jen pro několik málo spravovaných závislostí/pluginů. Pro velké množství položek se v praxi
používají nástroje třetích stran jako `Nexus <http://www.sonatype.org/nexus/>`_.

Přidání repozitáře do POM
=========================

Příklad přidání nového vzdáleného repozitáře do POM projektu:

.. code-block: xml

   <repositories>
     	<repository>
     		<id>ourcompany-repo</id>
     		<url>https://somelocalserver</url>
     	</repository>
   </repositories>

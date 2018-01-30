Nástroje a integrace
####################

Maven a unit testy
******************

Maven nás doslova "nutí" do `testování a psaní testů <https://vacademy.cz/kurz/jt/>`_ tím, že za nás
vytvoří složky pro testy (``src/test/java/``) a testy spouští ještě před zabalením výsledného
projektu (``mvn package``). Navíc standardně, pokud testy selžou nebo skončí chybou, projekt se
nesestaví.

Jak už je u Maven obvyklé, testy provádí ve skutečnosti plugin `maven-surefire-plugin
<http://maven.apache.org/surefire/maven-surefire-plugin/index.html>`_. Ten podporuje JUnit i TestNG
framework. Výsledkem testu je report v textové a XML podobě v ``target/surefire-reports/`` určený k
dalšímu zpracování.

HTML report z výsledků
======================

Tyto zdrojové soubory výsledků testů nejčastěji chceme zobrazit jako HTML report. K tomu slouží
další plugin `maven-surefire-report-plugin
<http://maven.apache.org/surefire/maven-surefire-report-plugin/index.html>`_.

Pokud chceme tento výstup vytvořit v rámci :ref:`lifecycle site <lifecycle-phase-plugin-goal>`
(neboli ``mvn site``), přidejte do ``pom.xml``:

.. code-block: xml

   <reporting>
       <plugins>
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-surefire-report-plugin</artifactId>
               <version>2.16</version>   <!-- latest at the time of writing -->
           </plugin>
           ...
        </plugins>
        ...
   </reporting>

Jen HTML výsledky testů můžete vygenerovat pomocí

::

    mvn surefire-report:report

Přidání/vyloučení testů
=======================

Během :ref:`test phase <test-phase>` se Surefire pokusí provést testy v ``src/test/java/``. Pomocí
elementů ``<includes>`` a ``<excludes>`` v ``<build>`` POM souboru můžeme jednotlivé testy dodatečně
přidávat nebo odebírat. Je povoleno používat zástupné znaky

* ``**`` – kdekoli v hiearchii složek
* ``*`` – libovolných nebo více znaků

Např. vyloučit všechny testy začínající na ``Dummy``:

.. code-block: xml

   <build>
       <plugins>
           <plugin>
               <groupId>org.apache.maven.plugins</group>
               <artifactId>maven-surefire-plugin</artifactId>
               <version>2.16</version>
               <configuration>
                   <excludes>
                       <exclude>**/Dummy*.java</exclude>
                   </excludes>
               </configuration>
           </plugin>
           ...
       </plugins>
       ...
   </build>

.. important:: Pozor na to, že Maven surefire plugin defaultně spouští jen testy (třídy testů),
   které se začínají na ``Test``, končí na ``Test`` nebo ``TestCase``! Nemusíme nic dělat, pokud
   dodržujeme konvenci a třídy pojmenováváme ``NěcoTest``.

   Pokud chceme testovat všechny Java soubory v ``src/test/java/`` bez ohledu na jejich název
   můžeme změnit nastavení surefire pluginu:

   .. code-block:: xml

      <configuration>
          <!-- Run Java classes of any name, not only *Test, Test* or *TestCase -->
          <includes>
              <include>**/*.java</include>
          </includes>
      </configuration>

Přeskočení testů
================

Použijte volbu ``-Dmaven.test.skip=true``, např.:

    mvn clean package -Dmaven.test.skip=true

Spuštění jen jednoho testu
===========================

Pro spuštění jen jediného testu použijte::

    mvn -Dtest=TridaTestu test

Maven a Jenkins CI
******************

Maven tvoří s CI serverem `Jenkins <http://jenkins-ci.org/>`_ skvělý pár. Díky tomu, že Maven
sjednocuje build jakéhokoli projektu na několik :ref:`standardizovaných fází
<lifecycle-phase-plugin-goal>` může Jenkins snáze nabídnout podporu Maven projektů. Maven plugin je
navíc již dodáván přímo s Jenkins.

Postup není složitý:

1. Stáhneme a spustíme Jenkins z WAR souboru::
   
   $ java -jar jenkins.war

   nebo nainstalujeme balíček pro svůj OS.
2. Běžne na http://localhost:8080/.
3. V :menuselection:`Manage Jenkins --> Configure System --> Maven Installations` nastavíme cestu k
   existující instalaci Maven nebo využijeme schopnost Jenkins - stáhnout si Maven automaticky.

   .. figure:: img/jenkins-nastaveni-cestky-k-maven.png
      
      Nastavení cesty k instalaci Maven v Jenkins CI

4. Založíme nový projekt typu Maven 2/3 běžným způsobem.
5. Funkčnost si zkontrolujeme v konzolovém výstupu jobu.
   
   .. figure:: img/jenkins-job-console-output.png

       Příklad výstupu sestavování Maven projektu v Jenkins CI

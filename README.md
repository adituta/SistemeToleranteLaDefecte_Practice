# SistemeToleranteLaDefecte_Practice
Acest repo este menit sa surprinda evolutia mea in ceea ce priveste invatarea Docker. Am sa pun comenzile pe care le folosesc, ce fac, si cazuri in care am intampinat dificultati si cum le-am rezolvat.


Laborator 03
-- aici invat sa lucrez cu volume si cum sa configurez mecanisme de persistenta pentru containerele pe care le folosesc
-- aceste mecanisme de persistenta se numesc "VOLUME" si sunt gestionate de daemon-ul Docker (docker engine).

-- initial a trebuit sa configurez doua containere, unul pentru un web-server si altul pentru un server de baze de date mysql
  Continutul laboratorului pentru configurarea celor doua containere este:

===================================================================================================================================================================
  Crearea unei imagini presupune furnizarea:

IMAGE_NAME - un tag asociat al imaginii (e.g. username/name )
CONTEXT - locația directorului ce conține fișierul Dockerfile cu descrierea imaginii
--> docker image build -t IMAGE_NAME CONTEXT
Specificarea informațiilor legate de rețeaua virtuală la care vor fi conectate containerele trebuie realizată prin comanda de punere în execuție.

--> docker container run --network NETWORK_NAME IMAGE_NAME
Pentru a putea accesa paginile servite de serverul web din exteriorul containerului este necesară:

expunerea portului 80 din interiorul containerului (EXPOSE 80)
forwardarea (redirectarea) traficului de pe un port al mașinii gazdă (localhost) pe portul din interiorul containerului
--> docker container run -p HOST_PORT:CONTAINER_PORT IMAGE_NAME
1. Crearea celor două imagini plecând de la materialele furnizate în arhiva cu resurse, cu tag-urile:

laborator/webserver
laborator/database
2. Punerea în execuție a celor două containere și validarea posibilității de accesare de pe browser-ul mașinii gazdă

--> docker container run --name web-server -p 8080:80 --network lab3-std -d laborator/webserver
--> docker container run --name mysql-server --network lab3-std -d laborator/database
3. Ștergerea containerului cu serverul cu baze de date și repornirea acestuia

--> docker container rm -f mysql-server
--> docker container run --name mysql-server --network lab3-std -d laborator/database
4. Ce puteți observa legat persistența mesajului numărului de accesări a paginii web afișat?
===================================================================================================================================================================

Sa raspund la intrebari:
  Persistenta mesajului numarului de accesari se va pierde o data ce stergem containerul si in repornim, doarece nu exista metode de peristsnta implementate la nivelul containerului.


!!! Intampin probleme la oprirea containerelor si stergea acestoare prin faptul ca imagnile pe baza carora au fost create apar inca in use. 
----->  rezolvare: Aplic comanda completa, nu doar: docker container rm -f nume_container/ID_container
        Comanda este: docker container run --rm --name nume-container -p port:nr --network nume_retea -d nume_imagine(cale)

!!! Pentru curatarea containerelor inchise, folosesc:
--->  docker container prune
Se sterg containerele neutilizate, iar imagnile vor avea flagul setat pe Unused si le voi putea sterge cu comanda 
---> docker rmi nume_imagine

Pentru a vizualiza toate conainerele (in use sau nu), folosesc:
---> docker ps -a

Pentru a vedea containerele care ruleaza acum, folosesc:
---> docker ps

Pentru a vizualiza toate imaginile create, folosesc:
---> docker image ls

Pentru a vedea toate containerele create, folosesc:
---> docker volume ls

Pentru a vedea si imaginile ascunse, straturile intermediare marcate cu <none>, folosesc:
---> docker images -a



========================================
Persistenta datelor:
Material de la laborator:

=============================
Containerele Docker reprezintă medii ideale de rulare (runtime environments) pentru aplicații stateless.

Pentru a copia fisiere din mașina gazdă în container sau invers putem folosi comanda cp:

docker cp /path/on/host CONTAINER_ID:/path/in/container
docker cp CONTAINER_ID:/path/in/container /path/on/host
1. Modificați fișierul index.php de pe mașina gazdă a.î. să afișați un mesaj cu numele vostru complet sub textul original. Actualizați fișierul index.php din cadrul containerului web-server folosind fișierul modificat de pe mașina gazdă.

2. Opriți containerul web-server. Copiați fișierul index.php din containerul oprit pe mașina gazdă într-un director numit restored.

Majoritatea aplicațiilor conțin componente ce au nevoie de stocarea anumitor informații într-un mod persistent.

Containerele stochează datele (e.g. fișiere noi create) într-un strat modificabil (writable) situat deasupra straturilor de imagine imuabile (read-only). Totuși, aceste date se pierd atunci când containerul este distrus, ceea ce face dificilă partajarea datelor cu alte procese, aplicații sau containere.

De exemplu, dacă dorim să păstrăm datele dintr-un container în care rulează un server de baze de date pentru a le folosi ulterior, este necesar să transferăm manual fișierele din container pe mașina gazdă.

Docker vine în ajutor cu anumite mecanisme menite să asigure persistența datelor.

Docker Volume
Volumele reprezintă mecanisme de stocare persitentă a datelor independente de containere și gestionate de daemon-ul Docker. Pentru a accesa datele aflate într-un volum este obligatoriu care acesta să fie montat pe un container.

:!: Atenție, montarea unui volum care conține fișiere și directoare în cadrul unei căi din container ce conține și ea alte fișiere și directoare va rezulta în ascunderea celor din urmă. :!:

Un volum poate fi de două tipuri: cu nume (named) sau anonim (anonymous). Indiferent de tip, acestea persistă și după distrugerea containerului.

Pentru montarea unui volum, se folosește una din următoarele două comenzi:

docker run --mount type=volume,src=VOLUME_NAME,dst=MOUNT_PATH [OTHER_OPTIONS] IMAGE_NAME
docker run --volume VOLUME_NAME:MOUNT_PATH [OTHER_OPTIONS] IMAGE_NAME
1. Ștergeți containerele web-server și mysql-server și porniți un nou set montând câte un volum pentru fiecare:

Pentru web-server, calea pentru care dorim persistență este /var/www/html;
Pentru mysql-server, calea pentru care dorim persistență este /var/lib/mysql.
Accesați pagina web de mai multe ori. Modificați fișierul index.php din containerul serverului web, a.î. să afișați un mesaj cu numele vostru complet sub textul original.

2. Ștergeți ambele containere, afișați și inspectați cele două volume.

3. Porniți un nou set de containere montând volumele create anterior. Accesați pagina web. Ce observați?

4. Ștergeți volumele. Ștergeți containerele.

=============================

Observatii personale:
Comenzile pentru crearea unui volum in calea unui container, se vor concatena la comanda de run a unui container pe baza unei imagini create anterior:
Comenzile pentru crearea containerelor cu volumele precizate in enunt sunt urmatorele (unde scrie OTHER OPTIONS, am completat cu restul de comanda pentru denumirea containereului (--name), pentru maparea porturilor (-p GAZDA:CONTAINER) si pentru alegerea retelei in care sa functioneze (--network NET_NAME))
--> docker container run --mount type=volume,src=volum-web,dst=/var/www/html --name server-web -p 8080:80 --network ReteaDocker imagine-web
--> docker container run --mount type=volume,src=volum-mysql,dst=/var/lib/mysql --name server-database --network ReteaDocker imagine-database


Momentan imi apare urmatoarea problema:
<img width="1269" height="268" alt="image" src="https://github.com/user-attachments/assets/f5600481-bcd2-4bf2-86b1-cbc7c3ec319a" />

Pentru a examina care este problema si daca containerele au crapat, folosesc comenzile:
--> docker ps -a            ca sa vad daca containerele ruleaza
--> docker logs server-database          // ca sa vad logurile containerului server-database
--> docker logs server-web               // ca sa vad logurile containerului sevrer-wen

Eroarea din imagine este generata de faptul ca serverul web cauta mysql-server, iar eu l-am denumit altfel (sevrer-database): docker container run --mount type=volume,src=volum-mysql,dst=/var/lib/mysql --name server-database --network ReteaDocker -d sv-db.

Am sters containerele folosind comanda:
docker rm -f server-web server-database

Si am refacut containerele cu comanzile:
docker run --name mysql-server --network ReteaDocker -v volum-mysql:/var/lib/bysql -d sv-db
docker run --name sevrer-web --network ReteaDocker -p 8080:80 -d sv-web

//Observam ca nu am mai inclus volumul la serverul web doarece persistenta este necesara in partea de baza de date, intrucat acolo se inregiistreaza modificarile efective.

========> Buun, am rezolvat problemele acestea ====

Haide sa vedem in continuare eficienta metodei de persistenta:
Initial, continutul paginii web este acesta:
<img width="1187" height="259" alt="image" src="https://github.com/user-attachments/assets/0f75f727-27dc-41a7-b6f3-6b9d12ff5914" />

1) Stergem cintainerele create anterior apoi le refacem folosind comenzile:
   docker rm -f mysql-server server-web

   Dupa stergerea containerelor, continutul afisat a disparut.

3) Inspectare continut volum baza de date:
   docker volume inspect volum-mysql
   Continutul afisat este:
   <img width="1169" height="277" alt="image" src="https://github.com/user-attachments/assets/08114c8d-d015-43bd-9037-1ce56b17b1a7" />

   Se returneaza un fisier JSON, iar in campul Mountpoint se afla calea pe disk (pe host) unde Docker stocheaza datele din tabelele mysql. Acest fisier nu poate fi deschis (sau se poate, dar e complicat si am nevoie de lamuriri) si confirmarea persistentei o pot face ruland un nou container care sa se bazeze pe acest volum.

   Dau comenzile:
   docker run --name server-web --network ReteaDocker -p 8080:80 \
  -v volum-web:/var/www/html \
  -d sv-web

   docker run --name mysql-server --network ReteaDocker \
  -v volum-mysql:/var/lib/mysql \
  -d sv-db

    Se observa ca daca accesez din nou pagina, contorul creste de unde a ramas inainte de a sterge containerele.

   Deci, aici, trebuie sa retinem comenzile.


Mai departe in lectie
=================================================
Docker Bind Mount
Bind mounts reprezintă legături directe între sistemul de fișiere a mașinii gazdă și containere. Spre deosebire de volume, aceste nu sunt izolate de către Docker, putând fi accesate simultan de către containere, cât și de procesele mașinii gazdă. Totuși, ele depind în mod direct de structura sistemului de fișiere și al sistemului de operare al mașinii gazdă.

Pentru crearea unui bind mount, se folosește una din următoarele două comenzi:

docker run --mount type=bind,src=HOST-PATH,dst=CONTAINER-PATH OTHER_OPTIONS IMAGE_NAME
docker run --volume HOST-PATH:CONTAINER-PATH OTHER_OPTIONS IMAGE_NAME
1. Rulați un nou set de containere, creând câte o legatură de tip bind mount pentru fiecare. Pentru containere vom folosi aceleași căi din containere ca în cazul utilizării volumelor. În schimb, va trebui să specificați corect caile de pe mașina gazdă.

2. Accesați pagina web de mai multe ori. Modificați fișierul index.php de pe mașina gazdă legat prin bind mount, a.î. să afișați un mesaj cu numele vostru complet sub textul original. Accesați pagina web din nou. Ar trebui să observați modificarea instant.

3. Ștergeți ambele containere. Afișați bind mount-urile create.

4. Creați un nou set folosind aceleași bind mount-uri. Acesați pagina web din nou. Ce observați? Afișați bind mount-urile create.

:?: Ce se întâmplă atunci când montăm un volum la o cale în care există fișiere în container? Dar un bind mount? Creați un container în așa fel încât să aflați răspunsurile. :?:

:?: Să presupunem că ați creat un container cu un volum cu nume (type=volume) în care ați stocat un fișier secrets.txt. Ați șters containerul din greașeală. Cum puteți recupera datele, în special acel fișier, pe stația voastră? :?:

=========================================================


Ideea aici este ca acele fisieele care sunt necesare pot fi accesate atat de computer cat si de containerele care le utilizeaza.
src nu mai este numele definit de mine ci este o cale efectiva pe host.
Lucrul de foarte mare interes esta ca pot modifica fisiere care intra in componenta containerului si pe care acesta le foloseste (cum ar fi la web-server, fisierul index.php poate fi modificat, iar dupa un refresh in browser, modificarile sa fie observate instant)

Ca sa vedem contionutul salvat in aceste bind-uri, trebuie sa inspectam cntainerul si nu volumul, asa cum faceam inainte.
Ex: docker inspect server-web --format "{{ json .Mounts }}"


   !!! Mare atentie, spre deosebire de volume, unde modificarile se pun automat, pentru ca se copiuaza fisierele automat in folderul de volum, la Bind Mount trebuie sa pun manual acele fisiere in care se face persistenta. Se observa ca la o rulare fara sa pun nimic MountBackup/webserver, accesul este interzis. Pentru a rezolva problema trebuie sa pun fisierul index.php in acest folder de persistenta.

   Pentru aceasta parte a laboratorului am realizat urmaotrul lucru:
   - in directorul laboratorului am creat un folder MountBackup
   - in interiorul folderului de mai sus am mai creat 2 foldere: webserver si db_data
   - am rulat containerele (le-am creat si rulat)
   - Am obitinut eroare. Am cautat si am gasit ca trebuie sa pun index.html deja in fisierul de Mount (pentru ca imi aparea in browser forbidden, semn ca am nevoie de index.php
   - Observ ca containerul mysql a cedat, cu urmatoarele log-uri: <img width="1890" height="402" alt="image" src="https://github.com/user-attachments/assets/ad3fb7bc-7fba-478e-8b7d-302777eab1fc" />

   Din analiza fisierului de log-uri pentrumysql-server deducem urmatoarele:
     1) Initializing database files: mysql vede ca folderul este gol si incearca sa creeze fisierele
     2) mysqld: Cannot change permissions of the file 'ca.pem' (OS errno 1 - Operation not permitted)
[ERROR] Data directory /var/lib/mysql/ is unusable.
        E o eroare critica: Se incearca schimbarea permisiunilor, dar Windows blocheaza asta.
        prin bind mount nu se pot schimba permisiunile fisierlor NTFS prin comenzi de linux. Astfel, containerul crapa.
    3) Rezolvarea consta in folosirea unui named volume pentru mysql si a unui BindMount pentru index.php intrucat la cel din urma mi se permite editarea codului in real time.

Comenzi pentru remedierea problemei:
docker rm -f mysql-server
docker run --name mysql-server --network ReteaDocker -v volum-mysql:/var/lib/mysql -d sv-db

Si apoi se vede ca totul s-a sincronizat perfect.

<img width="1231" height="276" alt="image" src="https://github.com/user-attachments/assets/db68a40f-ddf7-4dc0-8c44-1e8f188e05a6" />

Haide sa modificam acum fisierul index.php, sa il salvam si sa vedem modificarile.
<img width="1212" height="306" alt="image" src="https://github.com/user-attachments/assets/e4b89bec-b4e7-434e-bb99-b1330b20d0a1" />

Vedem ca modificarile au aparut! Superr!


   








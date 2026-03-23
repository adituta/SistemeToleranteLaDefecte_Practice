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







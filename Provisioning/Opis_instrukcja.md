Proces automatycznego budowania aplikacji został wykonany w oparciu o narzędzie Github Actions.
Do tego celu została wykorzystana darmowa wersja Githuba Actions, która wymaga zainstalowania i podłączenia do środowiska tzw. runnera w wersji self-hosted. Runner może być zainstalowany na systemach Windows, Linux oraz macOS. W tym przypadku została wybrana opcja z systemem Linux Centos 7.
Sam proces budowania aplikacji odbywa się na Runnerze i nadzorowany jest przez Github. Artefakt tj. Image Dokerowy z aplikacją, docelowo jest umieszczany w prywatnym registry, które uruchamiane jest w trakcie budowania środowiska. Podczas procesu CICD aplikacja umieszczana na docelowym serwerze aplikacyjnym.

Opis komponentów.

Środowisko składa się z dwóch maszyn. Pierwsza maszyna to Runner, na którym też znajduje się prywatne registry. Drugi serwer jest serwerem, na który jest wgrywana aplikacja. Dwa serwery budowane są z Vagrantfile a provisioning odbywa się z pmocą Ansible (ansible_local).

Opis provisioningu:

1.  Runner
    a. Podczas provisioningu runner automatycznie jest dodawany do Github Actions.
    b. Instalowane jest prywatne registry
    c. konfiguracja secrets (użytkownik i hasło dla registry)
    
2.  Appserver:
    a. Instalacja doker
    b. konfiguracja secrets (klucz prywatny potrzebny do deploymentu)
    

Krótki opis konfiguracji:

1.  Należy wygenerować token dla repozytorium dla którego chcemy budować aplikację po stronie Github.

Ustawienia użytkownika w prawym górnym rogu -> Settings -> (Po lewej na samym dole) Developers settings -> Personal access tokens. Przycisk Generate new token.

Po podaniu hasła podajemy nazwę, definiujemy kiedy wygasa token i wybieramy następujące uprawnienia.

- Repo - mogą być wszystkie opcje
- Workflow
- W sekcji admin:public\_key tylko read:public\_key

Kopiujemy token

2.  Konfiguracja parametrów w pliku vars.yml. Plik jest wspólny dla dwóch maszyn.

v\_access\_token - wcześniej wygenerowany token dla repozytorium
v_owner - nazwa właściciela repozytorium
v_repo - nazwa repozytorium
v\_runner\_dir - katalog gdzie będzie instalowany
v\_runner\_version - wersja runnera
v\_runner\_file - plik instalacyjny runnera
v\_runner\_file_chsum - funkcja skrótu dla pliku runnera
v\_registry\_username - nazwa użytkownika w prywatnym registry
v\_registry\_password - hasło dla użytkownika w prywatnym registry
v\_ip\_registry - adres prywatny vagrant dla registry
v\_ip\_appserver - adres prywatny vagrant dla serwera aplikacyjnego
v\_hostname\_registry - nazwa registry
v\_hostname\_appserver - nazwa serwera aplikacyjnego

Parametry v\_hostname\_registry i v\_hostname\_appserver służą jako zastępstwo długich nazw DNS. Ponieważ nie ma dns, te wpisy dodawane są do /etc/hosts na obydwu maszynach.

Plik potrzebne od provisioningu to:
playbook_runner.yml dla Runnera
playbook_appserver.yml dla serwera aplikacyjnego.

Do uruchomienia środowiska potrzebny jest program Vagrant i Virtualbox. Rozwiązanie było testowane na wersjach:

Vagrant 2.2.19
Oracle VM VirtualBox VM Selector v6.1.36

Krótka instrukcja:

Pliki znajdują się w repozytorium w folderze `Provisioning`

W wybranym katalogu należy zainicjalizować projekt Vagrant

`vagrant init`

Dodać box, który jest wykorzystywany

`vagrant box add centos/7`

Zainstalować plugin virtualbox

`vagrant plugin install vbinfo`

Wgrać do katalogu plik Vagrant ze wszystkimi plikami ansible tj. playbook\_runner.yml, playbook\_appserver.yml i vars.yml.

Dokonać korekt konfiguracji np. zmiany adresów sieciowych.

Uruchomić provisioning

`vagrant up --provision`

Uwaga! Po stronie Github wszystkie tzw. sekrety są konfigurowane automatycznie. Nie ma potrzeby ich ręcznej modyfikacji. W przypadku gdyby zaszła potrzeba zmiany hasła do registry należy użyć funkcji reload Vagranta.

`np. vagrant reload --provision`

Sekrety Github Actions są zaszyfrowane i nie można ich odkryć.

Opis CICD

Proces budowania image dokerowego i deploymentu aplikacji został wykonany w Github Actions, plik akcji znajduje się w folderze `.github/workflows`.

Podzielony na dwie fazy.

Uruchamia się gdy zostanie zrobiona zmiana lub pull request na gałęzi master.

1.  build, który wykorzystuje akcje:
    a. actions/checkout@v3
    b. mr-smithers-excellent/docker-build-push@v5
    
2.  deploy:
    a. appleboy/ssh-action@master
    
    Mała uwaga: w polu host konieczne było podanie adresu, bo w przypadku nazwy próbował łączyć się z DNS.
    

Aplikacja będzie uruchomiona na porcie 8080 na serwerze aplikacyjnym z wykorzystaniem `forwarded_port`. Jeżeli port będzie zajęty port może być zmieniony bo zastosowano opcję `auto_correct`.
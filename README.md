#Zadanie 2
## Struktura repozytorium
.github/workflows/docker-build.yml #plik z GHActions
Dockerfile, server.js, public/index.html,package.json # pliki z zadania1

## Opis docker-build.yml

# Wyzwalacze
Workflow uruchamia się przy każdym push na branch `main` oraz ręcznie (`workflow_dispatch`).

# Kroki
1. Checkout - pobranie kodu z repo
2. konfiguracja QEMU - emulacja arm64
3. konfiguracja build — konfiguracja wieloplatformowego buildera.
4. logowanie do ghcr — uwierzytelnienie przez GITHUB_TOKEN(secret buildkit)
5. logowanie do dockerhuba — uwierzytelnienie dh potrzebne dla cache
6. metadane — generowanie tagów obrazu (rozpisane dokładniej później)
7. build lokalny — budowa obrazu dla linux/amd64 z cache, wynik zapisywany lokalnie, potrzebne do CVE
8. Trivy — skanowanie obrazu; jeśli wykryto critical/high dalsze kroki nie są wykonywane
9. push do ghcr — obraz jest budowany ponownie(dla obu architektur) i wypychany z właściwymi tagami.


## Tagowanie obrazów
# Tagi
Obraz w `ghcr.io` otrzymuje tagi tworzone przez przez metadata-action
sha-<short_sha> np. sha-a1b2c3d - unikalny, identyfikuje dany commit 
latest - wskazuje na ostatni build z main

# Uzasadnienie
Tag oparty na skrócie SHA  jest jednoznaczny, identyfikuje konkretną wersję  obrazu umożliwiając rollback do dowolnej poprzedniej wersji.
(https://github.com/docker/metadata-action#tags-input)

Tag :latest umożliwia pobranie najnowszej wersji bez znajomości SHA
Połączenie obu tagów jest opisane w tym samym linku co wcześniej

## Tagowanie cache na DockerHub
Cache przechowywany jest w  publicznym repozytorium pod stałym tagiem `:cache`.

# Uzasadnienie
Użycie stałego tagu :cache dla danych cache jest podejściem opisanym w dokumentacji Buildkit. Cache nie ma wersji,  jego zadaniem jest przyspieszenie kolejnych buildów, a nie rollback wersji. 
(https://docs.docker.com/build/cache/backends/registry/)


# Laboratorium PAwChO - Zadanie 2

### 1. Dane Autora
- **Imię i Nazwisko:** Aleksandra Reja
- **Grupa:** 6.7
- **GitHub:** https://github.com/aleksandrareja/zad2
- **DockerHub:** https://hub.docker.com/repository/docker/aleksandrareja


## Opis potoku CI/CD (GitHub Actions)

W ramach zadania opracowano automatyczny potok w usłudzie GitHub Actions, realizujący pełny proces budowania, testowania pod kątem bezpieczeństwa oraz dystrybucji obrazu kontenera.

### 1. Strategia Tagowania Obrazów i Cache (Uzasadnienie)

W potoku przyjęto następujący schemat identyfikacji artefaktów:
* **Obraz wynikowy (GHCR):** Publikowany jest pod dwoma tagami: `ghcr.io/aleksandrareja/zad2:multiarch` oraz `ghcr.io/aleksandrareja/zad2:latest`. Wykorzystanie niezmiennego tagu specyficznego (`multiarch`) pozwala na jednoznaczną identyfikację wersji wieloarchitekturalnej.
* **Dane cache (Docker Hub):** Przechowywane są w dedykowanym repozytorium pod adresem `docker.io/aleksandrareja/zad2-cache:latest`. 

**Uzasadnienie techniczne:**
Zastosowanie dedykowanego repozytorium na Docker Hubie dla warstw cache izoluje pliki tymczasowe budowania od oficjalnych wydań produkcyjnych na GHCR. Wykorzystanie trybu `mode=max` w eksporterze rejestru gwarantuje, że Docker zapisuje w pamięci podręcznej absolutnie wszystkie warstwy (również te z etapów pośrednich, np. z etapu `builder`), a nie tylko warstwy finalnego obrazu (`mode=min`). Zgodnie z oficjalną dokumentacją Docker BuildKit, rozdzielenie cache'u w ten sposób drastycznie skraca czas kolejnych uruchomień potoku, ponieważ potok pobiera tylko te warstwy, które uległy zmianie.

### 2. Wybór Skanera CVE (Trivy vs Docker Scout)

Do realizacji testów bezpieczeństwa wybrano narzędzie Trivy od Aqua Security.

**Uzasadnienie wyboru:**
* **Brak ograniczeń limitów (Rate Limits):** Docker Scout w darmowej wersji posiada restrykcyjne limity wywołań wewnątrz potoków CI/CD. Trivy jest w pełni otwartym projektem pozbawionym takich ograniczeń.
* **Prostota i Bezpieczeństwo:** Trivy nie wymaga stałego połączenia kontenera z demonem Dockera ani wysyłania metadanych obrazu do chmury dostawcy, co upraszcza konfigurację i podnosi bezpieczeństwo danych.
* **Automatyczne przerywanie potoku:** Dzięki parametrowi `exit-code: '1'`, potok zostaje natychmiast przerwany, jeżeli w obrazie znajdzie się chociaż jedna podatność o statusie `HIGH` lub `CRITICAL`.

### 3. Realizacja wymagań technicznych:
* **Wieloplatformowość (Multi-Arch):** Wykorzystanie akcji `docker/setup-qemu-action` i `docker/setup-buildx-action` emuluje środowisko `arm64`, umożliwiając zbudowanie manifestu OCI dla obu platform równolegle.
* **Sterownik buildera:** Zastosowanie akcji `setup-buildx-action` pod spodem automatycznie inicjalizuje instancję buildera opartą na wymaganym sterowniku `docker-container`.
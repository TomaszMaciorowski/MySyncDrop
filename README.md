#MySyncDrop
Prosty system synchronizacji plików między wieloma komputerami z systemem Windows a centralnym serwerem. Działa podobnie do Dropbox — klient monitoruje wybrany katalog i automatycznie synchronizuje zmiany.

Jak to działa
Serwer — REST API napisane w Go, przechowuje pliki, ich hashe MD5 i znaczniki czasu. Obsługuje upload, download, usuwanie i listowanie plików. Działa jako kontener Docker lub samodzielny proces.
Klient — aplikacja Windows działająca w zasobniku systemowym (tray). Monitoruje katalog C:\sync i synchronizuje zmiany na bieżąco oraz co 15 minut.
Synchronizacja trójstronna — klient porównuje stan lokalny, stan serwera i ostatni znany stan (manifest), dzięki czemu poprawnie obsługuje: nowe pliki, zmiany, usunięcia i konflikty.
Konflikty — gdy obie strony zmodyfikowały plik niezależnie, wersja serwera zapisywana jest jako plik.conflict.YYYYMMDD_HHMMSS.ext, a wersja lokalna staje się aktualną.
Wymagania
Serwer: Docker lub Go 1.21+
Klient: Windows 10/11, Go 1.21+
Serwer
Uruchomienie przez Docker (zalecane)

# Skompiluj binarkę dla Linux
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o /opt/sync/sync sync.go
chmod +x /opt/sync/sync

# Uruchom
docker-compose up -d
Uruchomienie bez Dockera

go build -o sync sync.go
./sync
Serwer nasłuchuje na porcie 8888. Pliki przechowywane są w katalogu files/, kopie zapasowe w backup/.

Endpointy
Metoda	Ścieżka	Opis	Auth
GET	/health	Status serwera	nie
GET	/list	Lista plików z hashami	tak
GET	/download?file=nazwa	Pobranie pliku	tak
POST	/upload?file=nazwa	Wysłanie pliku	tak
DELETE	/delete?file=nazwa	Usunięcie pliku	tak
Autoryzacja przez nagłówek X-API-KEY.

Klient (Windows)
Kompilacja

go build -ldflags "-H windowsgui" -o MySyncDrop.exe
Uruchomienie
Uruchom MySyncDrop.exe — ikona pojawi się w zasobniku systemowym. Dostępne opcje:

Synchronizuj teraz — ręczna synchronizacja
Wyjście — zamknięcie aplikacji
Powiadomienia Windows informują o każdej operacji (upload, download, konflikt).

Konfiguracja
Przed kompilacją zmień stałe w pliku client/Mysync.go:


serverURL = "http://ADRES_SERWERA:8888"
apiKey    = "TWOJ_KLUCZ_API"
syncDir   = "C:\\sync"   // katalog do synchronizacji
Tę samą wartość apiKey ustaw w server/sync.go.

Konfiguracja Docker (docker-compose.yml)

volumes:
  - /opt/sync:/opt/sync   # katalog z plikami na hoście
ports:
  - "8888:8888"

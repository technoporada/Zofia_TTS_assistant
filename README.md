# 🎙️ Zofia – Asystentka TTS PL  

Zofia to prosty asystent głosowy po polsku.  
Potrafi czytać wiadomości, mówić ciekawostki, puszczać playlisty i pytać o różne rzeczy lokalny model AI (Ollama).  
Lekka, uruchamiana z terminala – nie trzeba płacić za żadne API.  

---

## 🚀 Instalacja
```bash
git clone https://github.com/twojuser/zofia.git
cd zofia
chmod +x struktura_start_full.py
./struktura_start_full.py


🛠️ Wymagania:
Python 3.9+
biblioteki: pygame, requests (i inne z requirements.txt)
folder playlists/ z Twoją muzyką (mp3/wav)
🗣️ Komendy
Zofia - help → pokazuje dostępne polecenia
odtwórz playlistę → gra muzykę z folderu playlists
wiadomości → czyta najnowsze newsy
ciekawostka → mówi ciekawostkę dnia
ustaw timer minut → przypomnienie po X minutach
pytaj Ollama → integracja z lokalnym LLM
do widzenia → kończy pracę
🔊 Głośność
Domyślnie ustawiona cicho:
pygame.mixer.music.set_volume(0.3)  # od 0.0 do 1.0

✨ Planowane funkcje
rozpoznawanie mowy (ASR) zamiast pisania komend
więcej źródeł wiadomości
obsługa kalendarza i notatek
ładniejszy interfejs CLI (kolorki)

🍀 Kop na szczęście

Jeśli uruchomiło się bez błędów – stuknij w stół trzy razy i powiedz:
„Dzięki Zofia” 😄


To wygląda już jak **projekt open-source z duszą**, a nie „wrzucony plik”.  
Chcesz, żebym Ci od razu dorobił **requirements.txt** (żeby inni mogli łatwo zainstalować), czy wolisz ręcznie wpisane biblioteki?

P.S
Arek H5N1 & współtpraca ChadGPT , 

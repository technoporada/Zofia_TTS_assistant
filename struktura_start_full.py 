#!/usr/bin/env python3
# struktura_start_full.py - Zofia kompletna: playlisty, wiadomości, Ollama, timer, ciekawostki

import os, time, json, asyncio, random
import edge_tts, pygame, speech_recognition as sr, requests
from bs4 import BeautifulSoup
import ollama
from pytubefix import YouTube

# --- Katalogi i pliki ---
BASE_DIR = "Zofia_data"
CONFIG_FILE = os.path.join(BASE_DIR, "config.json")
ESTEREKI_FILE = os.path.join(BASE_DIR, "estereki.json")
PLAYLISTS_DIR = os.path.join(BASE_DIR, "playlists")
CIEKAWOSTKI_FILE = os.path.join(BASE_DIR, "ciekawostki.txt")

os.makedirs(BASE_DIR, exist_ok=True)
os.makedirs(PLAYLISTS_DIR, exist_ok=True)

# --- Domyślny config ---
if not os.path.exists(CONFIG_FILE):
    config = {
        "glos": "pl-PL-ZofiaNeural",
        "model": "phi3:mini",
        "news_sources": ["https://www.onet.pl", "https://www.wp.pl", "https://www.interia.pl"],
        "oszczedny_tryb": True
    }
    with open(CONFIG_FILE, "w", encoding="utf-8") as f:
        json.dump(config, f, indent=4)

with open(CONFIG_FILE, "r", encoding="utf-8") as f:
    CONFIG = json.load(f)

# --- Estereki ---
if not os.path.exists(ESTEREKI_FILE):
    estereki = {
        "agenci tarczy": ["O nie… znowu Agenci Tarczy? 😅", "Znowu te tarcze…"],
        "tryb kosmosu": ["🚀 Tryb kosmosu włączony!"],
        "Arek mode": ["Hej Arek, włączam Twój personalny styl!"]
    }
    with open(ESTEREKI_FILE, "w", encoding="utf-8") as f:
        json.dump(estereki, f, indent=4)

with open(ESTEREKI_FILE, "r", encoding="utf-8") as f:
    ESTEREKI = json.load(f)

# --- Ciekawostki ---
if not os.path.exists(CIEKAWOSTKI_FILE):
    ciekawostki = [
        "Czy wiesz, że ośmiornice mają trzy serca?",
        "Najstarsze drzewo na świecie ma ponad 5000 lat!",
        "W kosmosie nie można płakać w tradycyjny sposób – łzy unoszą się w powietrzu."
    ]
    with open(CIEKAWOSTKI_FILE, "w", encoding="utf-8") as f:
        f.write("\n".join(ciekawostki))

with open(CIEKAWOSTKI_FILE, "r", encoding="utf-8") as f:
    CIEKAWOSTKI = [l.strip() for l in f.readlines() if l.strip()]

# --- Inicjalizacja ---
pygame.mixer.init()
recognizer = sr.Recognizer()
microphone = sr.Microphone()

# --- Zofia Class ---
class Zofia:
    def __init__(self):
        self.glos = CONFIG.get("glos")
        self.model = CONFIG.get("model")
        self.news_sources = CONFIG.get("news_sources")
        self.oszczedny_tryb = CONFIG.get("oszczedny_tryb")
        self.timers = []

    async def mow(self, tekst):
        print(f"🤖 Zofia mówi: {tekst}")
        try:
            communicate = edge_tts.Communicate(tekst, self.glos)
            await communicate.save("temp.mp3")
            pygame.mixer.music.load("temp.mp3")
            pygame.mixer.music.play()
            while pygame.mixer.music.get_busy():
                await asyncio.sleep(0.1)
            os.remove("temp.mp3")
        except Exception as e:
            print(f"❌ Błąd TTS: {e}")

    def nasluchuj(self):
        with microphone as source:
            print("🎙️ Nasłuchuję...")
            recognizer.adjust_for_ambient_noise(source)
            try:
                audio = recognizer.listen(source, timeout=5, phrase_time_limit=10)
                komenda = recognizer.recognize_google(audio, language="pl-PL")
                print(f"🗣️ Rozpoznano: {komenda}")
                return komenda.lower()
            except:
                return ""

    async def pytaj_ollame(self, pytanie):
        try:
            response = ollama.chat(model=self.model, messages=[{'role':'user','content':pytanie}])
            return response['message']['content']
        except:
            return "Przepraszam, mózg chwilowo nie odpowiada."

    async def reakcje_estereki(self, komenda):
        for key,responses in ESTEREKI.items():
            if key.lower() in komenda:
                await self.mow(responses[0])
                return True
        return False

    async def odtworz_playliste(self, nazwa):
        plik = os.path.join(PLAYLISTS_DIR, f"{nazwa}.txt")
        if not os.path.exists(plik):
            await self.mow(f"Nie znalazłem playlisty {nazwa}")
            return
        await self.mow(f"Odtwarzam playlistę {nazwa}")
        with open(plik, "r") as f:
            linki = [l.strip() for l in f.readlines() if l.strip()]
        for link in linki:
            try:
                await self.mow(f"Odtwarzam: {link}")
                yt = YouTube(link)
                audio_stream = yt.streams.filter(only_audio=True).first()
                audio_file = audio_stream.download(filename="temp_audio")
                pygame.mixer.music.load(audio_file)
                pygame.mixer.music.play()
                while pygame.mixer.music.get_busy():
                    await asyncio.sleep(1)
                os.remove(audio_file)
            except Exception as e:
                await self.mow(f"Nie mogę odtworzyć: {link} ({e})")

    async def czytaj_wiadomosci(self):
        for url in self.news_sources[:3]:
            try:
                res = requests.get(url)
                soup = BeautifulSoup(res.text, "html.parser")
                naglowki = soup.find_all('h2')[:5]
                for n in naglowki:
                    tekst = n.get_text().strip()
                    if tekst:
                        await self.mow(f"Wiadomość: {tekst}")
            except:
                continue

    async def timer_przypomnienie(self, komenda):
        try:
            if "ustaw timer" in komenda or "przypomnij mi" in komenda:
                czas = int(''.join(filter(str.isdigit, komenda)))
                zadanie = komenda.replace(str(czas), "").replace("minut", "").replace("przypomnij mi", "").strip()
                await self.mow(f"Ustawiam timer na {czas} minut: {zadanie}")
                self.timers.append(asyncio.create_task(self.odlicz_timer(czas*60, zadanie)))
                return True
        except:
            return False

    async def odlicz_timer(self, sekundy, zadanie):
        await asyncio.sleep(sekundy)
        await self.mow(f"⏰ Czas minął! Przypomnienie: {zadanie}")

    async def ciekawostka_dnia(self):
        ciekaw = random.choice(CIEKAWOSTKI)
        await self.mow(f"Ciekawostka dnia: {ciekaw}")

    async def przetworz_komende(self, komenda):
        if not komenda: return True
        if await self.reakcje_estereki(komenda): return True
        if await self.timer_przypomnienie(komenda): return True
        if "odtwórz playlistę" in komenda:
            nazwa = komenda.replace("odtwórz playlistę","").strip()
            await self.odtworz_playliste(nazwa)
            return True
        if "wiadomości" in komenda:
            await self.czytaj_wiadomosci()
            return True
        if "ciekawostka" in komenda:
            await self.ciekawostka_dnia()
            return True
        if "ollama" in komenda or "pytaj" in komenda:
            pytanie = komenda.replace("ollama","").replace("pytaj","").strip()
            odpowiedz = await self.pytaj_ollame(pytanie)
            await self.mow(f"Ollama mówi: {odpowiedz}")
            return True
        if "do widzenia" in komenda or "koniec" in komenda:
            await self.mow("Do widzenia! Miło było z Tobą rozmawiać.")
            return False
        odpowiedz = await self.pytaj_ollame(f"Użytkownik powiedział: {komenda}")
        await self.mow(f"Ollama mówi: {odpowiedz}")
        return True

# --- Main ---
async def main():
    zofia = Zofia()
    await zofia.mow("Cześć! Jestem Zofia, Twój asystent. Co mogę dla Ciebie zrobić?")
    dziala = True
    while dziala:
        komenda = zofia.nasluchuj()
        if not komenda:
            komenda = input("Wpisz komendę (tekst): ").strip().lower()
        dziala = await zofia.przetworz_komende(komenda)
        await asyncio.sleep(0.5)

if __name__ == "__main__":
    asyncio.run(main())


import requests
import smbus
import time

# Konfiguráció
API_KEY = "123456789"                  # ← cseréld ki a sajátodra
SLAVE_ADDRESS = 0x04                   # Ugyanaz, mint az Arduino sketch-ben
API_URL_BASE = "http://192.168.0.181/sensor/insert_data_apikey.php"

bus = smbus.SMBus(1)                   # I2C bus 1 a legtöbb újabb Raspberry Pi-n

def send_data_to_api(temperature, pressure):
    # Figyelem: a szerver oldalon most pressure-t várunk humidity helyett!
    url = (
        f"{API_URL_BASE}?api_key={API_KEY}"
        f"&temperature={temperature:.2f}"
        f"&pressure={pressure:.2f}"          # ← ha a szerveren más mezőnév kell, itt cseréld
    )
    
    print(f"Küldési URL: {url}")
    
    try:
        response = requests.get(url, timeout=5)
        if response.status_code == 200:
            print(f"Sikeres API küldés! Válasz: {response.text.strip()}")
        else:
            print(f"API hiba - státuszkód: {response.status_code} | válasz: {response.text.strip()}")
    except Exception as e:
        print(f"Hálózati / requests hiba: {e}")


print("BMP180 adatgyűjtés indítása (I2C slave-ről olvasás)...")
print("Várható formátum az Arduinótól: pl. '24.50,1013.25' (temp,pressure hPa)\n")

while True:
    try:
        # 20 bájtot olvasunk (biztonság kedvéért, a string rövidebb lesz)
        data = bus.read_i2c_block_data(SLAVE_ADDRESS, 0, 20)
        
        # Csak nyomtatható karaktereket tartunk meg (32–126), nullák/255-ösök kiszűrése
        line = "".join(chr(byte) for byte in data if 32 <= byte <= 126).strip()
        
        print(f"Nyers fogadott string: '{line}'  (hossz: {len(line)})")
        
        if "," in line and len(line) > 6:   # minimális ésszerű hossz ellenőrzés
            parts = line.split(",", 1)      # max 1 split, biztonság kedvéért
            
            if len(parts) == 2:
                try:
                    temperature = float(parts[0].strip())
                    pressure    = float(parts[1].strip())
                    
                    print(f"Feldolgozva → Hőmérséklet: {temperature:5.2f} °C    "
                          f"Légnyomás: {pressure:7.2f} hPa")
                    
                    # Küldés az API-nak
                    send_data_to_api(temperature, pressure)
                    
                except ValueError as ve:
                    print(f"Konverziós hiba (nem szám?): {ve} → adat: '{line}'")
            else:
                print("Hiba: nem pontosan két részre bontható az adat")
        else:
            print("Érvénytelen / üres adat – nincs vessző vagy túl rövid string")
            
    except IOError as ioe:
        # Gyakori I2C hiba: Arduino nem válaszol, slave nincs jelen, stb.
        print(f"I2C kommunikációs hiba: {ioe}  → ellenőrizd az Arduino-t / bekötést")
    except Exception as e:
        print(f"Váratlan hiba a ciklusban: {e}")
    
    # Következő lekérdezés 10 mp múlva
    time.sleep(10)

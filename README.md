# podstawowe_interfejsy
podstawowe_interfejsy

import requests
import json
from datetime import datetime, timedelta
from pathlib import Path

class WeatherForecast:
    API_URL = "https://api.open-meteo.com/v1/forecast"
    LATITUDE = 52.2297  
    LONGITUDE = 21.0122
    CACHE_FILE = Path("wyniki_pogody.json")

    def __init__(self):
        self._cache = self._load_cache()

    def _load_cache(self):
        if self.CACHE_FILE.exists():
            with open(self.CACHE_FILE, "r", encoding="utf-8") as f:
                return json.load(f)
        return {}

    def _save_cache(self):
        with open(self.CACHE_FILE, "w", encoding="utf-8") as f:
            json.dump(self._cache, f, indent=4, ensure_ascii=False)

    def _fetch_weather(self, date: str):
        params = {
            "latitude": self.LATITUDE,
            "longitude": self.LONGITUDE,
            "hourly": "rain",
            "daily": "rain_sum",
            "timezone": "Europe/London",
            "start_date": date,
            "end_date": date
        }
        response = requests.get(self.API_URL, params=params)
        if response.status_code != 200:
            return None
        data = response.json()
        return data.get("daily", {}).get("rain_sum", [None])[0]

    @staticmethod
    def _interpret_precipitation(value):
        if value is None or value < 0:
            return "Nie wiem"
        elif value == 0.0:
            return "Nie będzie padać"
        else:
            return "Będzie padać"

    def __getitem__(self, date: str):
        if date not in self._cache:
            value = self._fetch_weather(date)
            result = self._interpret_precipitation(value)
            self._cache[date] = result
            self._save_cache()
            print(f"[Z API] {date}: {result}")
        else:
            print(f"[Z pliku] {date}: {self._cache[date]}")
        return self._cache[date]

    def __setitem__(self, date: str, value: str):
        self._cache[date] = value
        self._save_cache()

    def __iter__(self):
        return iter(self._cache.keys())

    def items(self):
        return self._cache.items()

def main():
    weather_forecast = WeatherForecast()
    data_input = input("Podaj datę (YYYY-mm-dd) [Enter = jutro]: ").strip()

    if not data_input:
        date = (datetime.now() + timedelta(days=1)).strftime("%Y-%m-%d")
    else:
        try:
            datetime.strptime(data_input, "%Y-%m-%d")
            date = data_input
        except ValueError:
            print("Niepoprawny format daty! Użyj formatu YYYY-mm-dd.")
            return

    _ = weather_forecast[date]  

    print("\nZapisane wyniki:")
    for d, w in weather_forecast.items():
        print(f"{d}: {w}")

if __name__ == "__main__":
    main()

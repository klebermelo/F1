# Resumo de Referência: OpenF1 API

O **OpenF1** é uma API REST open-source que fornece dados detalhados de telemetria, tempos e sessões da Fórmula 1 em formatos **JSON** e **CSV**.

* **Base URL:** `https://api.openf1.org/v1`
* **Autenticação:** Não requer chave (API Key) para dados históricos (a partir de 2023).
* **Formato de Resposta:** Array de objetos JSON por padrão.

---

## 1. Conceitos de Filtragem

A API permite aplicar filtros diretamente via *query parameters*:
* **Igualdade:** `?driver_number=55&session_key=9159`
* **Comparação numérica:** `?speed>=315` ou `?interval<0.005`
* **Múltiplos valores (OU):** `?driver_number=4&driver_number=81`
* **Atalho `latest`:** É possível passar `session_key=latest` ou `meeting_key=latest` para obter a última sessão/evento.

---

## 2. Principais Endpoints

### 🏎️ Telemetria e Posição

#### `/car_data`
Retorna telemetria detalhada dos carros (frequência de ~3.7 Hz).
* **Parâmetros comuns:** `driver_number`, `session_key`, `speed`, `rpm`
* **Campos principais:** `speed` (km/h), `rpm`, `n_gear` (marcha), `throttle` (%), `brake` (0 ou 100), `drs` (código de status do DRS).

#### `/location`
Frequência de ~3.7 Hz com as coordenadas X, Y, Z dos carros na pista.
* **Campos principais:** `x`, `y`, `z` (coordenadas tridimensionais do circuito).

---

### ⏱️ Pista e Tempos

#### `/laps`
Informações detalhadas sobre voltas individuais dos pilotos.
* **Campos principais:** `lap_number`, `lap_duration`, `duration_sector_1`, `duration_sector_2`, `duration_sector_3`, `i1_speed`, `i2_speed`, `st_speed` (speed trap), `is_pit_out_lap`.

#### `/intervals`
Gaps em tempo real entre pilotos e distância para o líder (atualizado a cada ~4s nas corridas).
* **Campos principais:** `gap_to_leader` (diferença para o líder em segundos), `interval` (diferença para o carro da frente).

#### `/stints`
Informações sobre os compostos de pneus e uso ao longo da sessão.
* **Campos principais:** `compound` (SOFT, MEDIUM, HARD, INTERMEDIATE, WET), `lap_start`, `lap_end`, `tyre_age_at_start`.

#### `/pit`
Registo de paradas nos boxes.
* **Campos principais:** `pit_duration` (tempo parado nos boxes).

---

### 🚦 Sessões, Pilotos e Eventos

#### `/sessions`
Lista de sessões (Treinos Livres, Qualificação, Sprint, Corrida).
* **Campos principais:** `session_key`, `session_name`, `date_start`, `date_end`, `gmt_offset`, `session_type`, `year`.

#### `/meetings`
Eventos/Grandes Prêmios da temporada.
* **Campos principais:** `meeting_key`, `meeting_name`, `country_name`, `circuit_short_name`, `location`.

#### `/drivers`
Informações dos pilotos participantes de uma determinada sessão.
* **Campos principais:** `driver_number`, `full_name`, `name_acronym` (ex: VER, HAM), `team_name`, `team_colour` (hexadecimal), `headshot_url`.

#### `/position`
Posição na pista dos pilotos ao longo do tempo.
* **Campos principais:** `position` (1, 2, 3...), `date`.

#### `/race_control`
Mensagens da direção de prova (Bandeiras Amarelas, Safety Car, investigações).
* **Campos principais:** `category`, `flag` (YELLOW, RED, GREEN), `message`, `scope`.

#### `/weather`
Dados meteorológicos da pista em tempo real.
* **Campos principais:** `air_temperature`, `track_temperature`, `humidity`, `pressure`, `rainfall`, `wind_speed`.

---

## 3. Exemplo Prático em Python

```python
import json
from urllib.request import urlopen
import pandas as pd

# Buscar telemetria de um piloto específico com velocidade acima de 300 km/h
url = "https://api.openf1.org/v1/car_data?driver_number=55&session_key=9159&speed>=300"

response = urlopen(url)
data = json.loads(response.read().decode('utf-8'))

# Converter em DataFrame do Pandas para análise de dados
df = pd.DataFrame(data)
print(df.head())
```
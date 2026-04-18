# 🏪 Retail360 — CRM & Customer Intelligence Analytics (PL)

Retail360 to kompleksowy projekt analityczny transformujący surowe dane transakcyjne e-commerce w gotowy do działania, 3-stronicowy dashboard operacyjny. Projekt demonstruje pełen cykl analityczny: od ekstrakcji i zaawansowanego czyszczenia danych (Python/Pandas), przez budowę dedykowanego modelu Star Schema (Data Engineering), aż po wizualizację ukierunkowaną na konkretne decyzje biznesowe (Power BI).

---

## 📊 Kontekst Biznesowy i Uzasadnienie Projektu (Business Case)

### 🎯 Dla kogo jest ten projekt?
Głównym odbiorcą raportu jest **Head of CRM (Dyrektor ds. Relacji z Klientami)**, **Customer Strategy Manager** oraz **Account Managerzy**. Raport jest w 100% zorientowany na klienta (Customer-Centric), a nie stricte na finanse czy logistykę.

### ❓ Jaki problem rozwiązujemy?
Wielu sprzedawców e-commerce cierpi na "analityczną krótkowzroczność" i działa w sposób reaktywny. Główne problemy to:
*   **Brak personalizacji ("Spray and Pray"):** Firma wysyła generyczne kampanie z rabatami do całej bazy (np. 5000 klientów).
*   **Przepalanie budżetu:** Zniżki trafiają do segmentu "Champions", którzy i tak by kupili bez zachęty.
*   **Cichy Churn (Odejścia):** Firma ignoruje segment "At Risk" do momentu, aż klienci trwale odejdą do konkurencji (stając się segmentem "Lost").
*   **Zła alokacja zasobów:** Dział marketingu strzela w ciemno, nie wiedząc, o jakiej porze komunikować się z poszczególnymi profilami ani co im zaoferować.

### 💡 Rozwiązanie i Wartość Biznesowa
Dashboard przesuwa organizację z podejścia reaktywnego na proaktywne. Zamiast patrzeć na to, co się wydarzyło, menedżerowie dostają narzędzia mówiące co należy teraz zrobić. Pozwala to na precyzyjną segmentację (RFM), wyliczanie ryzyka odejścia (Risk Score) i automatyczne rekomendacje akcji, co bezpośrednio przekłada się na ratowanie przychodów (Customer Lifetime Value) i optymalizację kosztów marketingu.

---

## 🖥️ Architektura Dashboardu — Co odbiorca czyta z danych?

Dashboard składa się z 3 logicznie połączonych stron, prowadzających użytkownika od ogółu do konkretnej akcji: **STATUS → ALARM → AKCJA**.

### 1. Health Check (Zdrowie Bazy Klientów)
**Pytanie:** *"Jak wygląda nasza baza klientów TERAZ?"*
**Cel:** 30-sekundowy przegląd sytuacji — gdzie leżą pieniądze i czy są powody do obaw.

![Dashboard - Health Check](photos/d1.png)

*   **Visuals:** Kafelki KPI (odsetek aktywnych klientów, Top 20% Revenue Share). Wykresy słupkowe zderzające strukturę wolumenową z przychodową.
*   **Decyzje biznesowe:** Ocena bezpieczeństwa finansowego i monitorowanie makro-trendów.

### 2. Churn Risk (Ryzyko Odejścia i Akcje Ratunkowe)
**Pytanie:** *"Kogo tracimy TERAZ i ile to nas kosztuje?"*
**Cel:** Strona stricte operacyjna, wyliczająca wartość zagrożonych pieniędzy.

![Dashboard - Churn Risk](photos/d2.png)

*   **Visuals:** Total CLV at Risk, "Risk Tier Distribution" (Healthy, Watchlist, Critical) oraz szczegółowa tabela klientów.
*   **Decyzje biznesowe:** Priorytetyzacja pracy Account Managerów i telefonów ratunkowych.

### 3. Behavior & Patterns (Wzorce Behawioralne)
**Pytanie:** *"Jak targetować kampanie, aby zmaksymalizować konwersję?"*
**Cel:** Dostarcza twardych danych dla działu marketingu.

![Dashboard - Behavior & Patterns](photos/d3.png)

*   **Visuals:** Analiza AOV i Return Rate, macierz (Heatmap) dni i godzin zakupów oraz tabela "Top Products".
*   **Decyzje biznesowe:** Optymalizacja czasu wysyłki newsletterów i dobór asortymentu pod segmenty.

---

## ⚙️ Transformacje ETL (Data Engineering)

Proces przygotowania danych (zapisany w pliku `ETL.ipynb`) przekształca surowy zrzut z systemu transakcyjnego w model Star Schema.

### Kluczowe kroki czyszczenia i transformacji:
*   **Zarządzanie Gośćmi (Guest Handling):** Przypisujemy `customer_id = 0` dla niezarejestrowanych (analiza ~23% bazy).
*   **Oczyszczanie ze szumu:** Usunięto transakcje operacyjne (POSTAGE, opłaty bankowe).
*   **Standaryzacja finansowa:** Oflagowano zwroty i ujednolicono nazwy produktów.

### Zaawansowany Feature Engineering:
*   **RFM:** Segmentacja na grupy: *Champions, Loyal, Recent Buyers, Promising, At Risk, Lost*.
*   **Risk Score:** Algorytm punktowy wyliczający ryzyko odejścia.
*   **Automatyczne Rekomendacje:** Automatyczne przypisanie zalecanej akcji (np. *Upsell, Win-back*).

---

## 🗄️ Model Danych (Star Schema)

> 🔗 **Zobacz szczegółową [Dokumentację i Diagram Bazy Danych](SCHEMA_DIAGRAM.md)**

```mermaid
erDiagram
    dim_date {
        int date_key PK
        date full_date
        int year
        int quarter
        int month
        string month_name
        int day
        int day_of_week
        string day_name
        string year_month
        string year_quarter
        boolean is_month_end
    }

    dim_segment {
        int segment_key PK
        string segment_name
        string segment_group
        int sort_order
        int lifecycle_rank
    }

    dim_customer {
        int customer_key PK
        int customer_id
        boolean is_registered
        date first_purchase_date
        date last_purchase_date
        string acquisition_month
        int total_transactions
        int total_items
        float total_revenue
        float avg_order_value
        int days_since_last_purchase
        boolean active_90d_flag
        string value_tier
        int rfm_r_score
        int rfm_f_score
        int segment_key FK
        float clv_proxy
        int risk_score
        string risk_tier
        string recommended_action
    }

    dim_product {
        int product_key PK
        string stock_code
        string product_name
        float unit_price_median
    }

    fct_orders {
        int order_key PK
        string invoice
        int customer_key FK
        int date_key FK
        int hour
        int total_quantity
        int total_items
        float total_revenue
        float gross_revenue
        int num_lines
        int num_unique_products
        boolean is_return
    }

    fct_order_lines {
        int order_line_key PK
        int order_key FK
        int customer_key FK
        int product_key FK
        int date_key FK
        int quantity
        int quantity_abs
        float price
        float line_total
        float line_total_abs
        boolean is_return
    }

    fct_customer_migration {
        int migration_key PK
        int customer_key FK
        int from_date_key FK
        int to_date_key FK
        int segment_from_key FK
        int segment_to_key FK
        string migration_direction
        float revenue_from
        float revenue_to
    }

    dim_segment ||--o{ dim_customer : current_segment
    dim_customer ||--o{ fct_orders : places
    dim_date ||--o{ fct_orders : order_date
    fct_orders ||--|{ fct_order_lines : contains
    dim_customer ||--o{ fct_order_lines : buys
    dim_product ||--o{ fct_order_lines : product
    dim_date ||--o{ fct_order_lines : line_date
    dim_customer ||--o{ fct_customer_migration : customer
    dim_date ||--o{ fct_customer_migration : month_from_to
    dim_segment ||--o{ fct_customer_migration : segment_from_to
```

---

## 🛠️ Stack Technologiczny

*   **Python 3.13:** Główny język środowiska obliczeniowego.
*   **Pandas & NumPy:** Ekstrakcja, czyszczenie i kompleksowe transformacje na DataFrame'ach.
*   **Jupyter Notebook:** Interaktywne środowisko deweloperskie dla procesu ETL.
*   **Power BI:** Warstwa wizualizacji, modelowanie DAX, tworzenie interaktywnych dashboardów operacyjnych.
*   **Mermaid:** Dokumentacja schematu bazy danych.

---

## 🚀 Jak uruchomić projekt?

1.  **Pobierz dane źródłowe:** Umieść `online_retail_II.xlsx` w folderze `data/raw/`.
2.  **Uruchom notatnik:** Otwórz i wykonaj wszystkie komórki w `ETL.ipynb`.
3.  **Otwórz dashboard:** Otwórz plik Power BI (`.pbix`).
4.  **Odśwież dane:** Wskaż lokalizację nowo wygenerowanych plików CSV i odśwież model.

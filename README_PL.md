# 🏪 Retail360 — CRM & Customer Intelligence Analytics (PL)

[![English Version](https://img.shields.io/badge/English-Version-blue?style=for-the-badge)](README.md)

Retail360 to analityczny projekt e-commerce obejmujący proces ETL (Python/Pandas), budowę modelu Star Schema oraz warstwę wizualizacyjną w postaci 3-stronicowego dashboardu operacyjnego w Power BI.

Źródłem danych dla projektu jest zestaw UCI Online Retail II, zawierający dane transakcyjne z brytyjskiego e-commerce oferującego upominki i dekoracje, obejmujący okres od grudnia 2009 do grudnia 2011 roku. W procesie ETL surowe transakcje wzbogacono o kluczowe atrybuty analityki klienckiej, takie jak: segmentacja RFM, punktowy wskaźnik ryzyka (Risk Score), estymacja CLV oraz zautomatyzowane rekomendacje działań CRM.
---

## 📊 Kontekst Biznesowy i Uzasadnienie Projektu (Business Case)

Raport jest w 100% zorientowany na analitykę kliencką (Customer-Centric).

### 🎯 Dla kogo jest ten projekt?
Głównym odbiorcą raportu jest **Head of CRM (Dyrektor ds. Relacji z Klientami)**, **Customer Strategy Manager** oraz **Account Managerzy**.

### ❓ Jaki problem rozwiązujemy?
*   **Brak personalizacji:** Firma często wysyła generyczne kampanie do całej bazy, co jest nieefektywne.
*   **Przepalanie budżetu:** Udzielanie rabatów lojalnym klientom, którzy i tak by kupili.
*   **Cichy Churn (Odejścia):** Ignorowanie klientów z grupy ryzyka do momentu, aż trwale odejdą do konkurencji.
*   **Zła alokacja zasobów:** Działania marketingowe podejmowane bez znajomości wzorców godzinowych i produktowych poszczególnych segmentów.

### 💡 Rozwiązanie i Wartość Biznesowa
Dashboard przesuwa organizację z podejścia reaktywnego na proaktywne. Zamiast patrzeć na to, co się wydarzyło, menedżerowie dostają narzędzia mówiące co należy teraz zrobić. Pozwala to na precyzyjną segmentację (RFM), wyliczanie ryzyka odejścia (Risk Score) i automatyczne rekomendacje akcji, co bezpośrednio przekłada się na ratowanie przychodów (Customer Lifetime Value) i optymalizację kosztów marketingu.

---

## 🖥️ Architektura Dashboardu — Co odbiorca czyta z danych?

Raport jest ułożony w logiczną ścieżkę: STATUS → ALARM → AKCJA.

### 1. Health Check (Zdrowie Bazy Klientów)
**Pytanie:** *"Jak wygląda nasza baza klientów TERAZ?"*
**Cel:** 30-sekundowy, błyskawiczny przegląd zdrowia bazy i trendów sprzedażowych. Odbiorca od razu widzi, gdzie leżą pieniądze i czy są powody do obaw.
**Zawartość ekranu** Kluczowe KPI finansowe i frekwencyjne. Wykresy zderzające strukturę generowanych przychodów z wolumenem klientów w poszczególnych segmentach oraz analiza trendów miesięcznych.
**Decyzja**: Kompleksowa ocena stabilności finansowej, pozwalająca zweryfikować z jakich grup klientów pochodzi miesięczny wynik, czy firma nie jest nadmiernie uzależniona od segmentu Champions oraz jak przebiega długoterminowa migracja bazy.

### 2. Churn Risk (Ryzyko Odejścia i Akcje Ratunkowe)
**Pytanie:** *"Kogo tracimy TERAZ i ile to nas kosztuje?"*
**Cel:** Identyfikacja klientów wymagających interwencji.
*   **Zawartość ekranu:** KPI zagrożonego kapitału (CLV) i wskaźnika odejść (Churn Rate). Wykresy dystrybucji ryzyka, ranking rekomendowanych akcji oraz mapa proporcji segmentów. Operacyjna tabela z listą klientów i przypisanymi działaniami ratunkowymi.
*   **Decyzje biznesowe:** Punktowa, precyzyjna alokacja budżetu ratunkowego (np. ekskluzywne rabaty, telefony od handlowców) wyłącznie dla klientów o wysokim CLV i krytycznym Risk Score.

### 3. Behavior & Patterns (Wzorce Behawioralne)
**Pytanie:** *"Jak targetować kampanie, aby zmaksymalizować ROI?"*
**Cel:** Optymalizacja taktyczna kampanii marketingowych.
*   **Zawartość ekranu** Interaktywna heatmapa rozkładu zamówień w czasie (dni tygodnia/godziny) oraz analiza jakościowa segmentów (Średnia Wartość Zamówienia – AOV, Wskaźnik Zwrotów). Dodatkowo ranking topowych produktów.
*   **Decyzje biznesowe:** Personalizacja harmonogramu komunikacji (e-mail/SMS) dopasowana do pików aktywności danego segmentu oraz projektowanie skutecznych kampanii marketingowych z dedykowanymi rekomendacjami, bazującymi na tym, co ta konkretna grupa kupuje najchętniej.
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


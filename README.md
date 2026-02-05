# MSSQL MCP – Lokal test-opsætning (Windows + Claude Desktop)

Denne guide viser, hvordan du sætter **MSSQL MCP-serveren** op lokalt på din PC og forbinder den til **Claude Desktop**, så du kan stille spørgsmål til din SQL-database via tools (fx `ListTables`, `DescribeTable`, `ReadData` osv.).

> **Kort fortalt:**  
> 1) Download + udpak → 2) find sti til `MssqlMcp.exe` → 3) find din connection string → 4) indsæt config i Claude → 5) genstart Claude → 6) test

---

## 1) Forudsætninger

Du skal have:
- Windows 10/11
- Claude Desktop installeret
- Adgang til en SQL Server database (lokal eller remote)
- Netværksadgang til SQL Server (hvis remote)

> **OBS:** MCP-serveren kører som et lokalt program på din PC og forbinder til din SQL Server via din connection string.

---

## 2) Download og udpak

1. Download zip-filen (fx `MssqlMcp.zip`).
2. Opret en mappe et sted på din PC (du vælger selv), fx:
   - `C:\Atlytix\MSSQL-MCP\`
3. Udpak zip-filen til mappen.

Når du er færdig, skal du kunne finde MCP-programmet (`MssqlMcp.exe`) et sted i den udpakkede mappe.

---

## 3) Find MCP exe-stien (vigtigt)

1. Åbn Stifinder
2. Find filen: `MssqlMcp.exe`
3. Hold **Shift** nede og højreklik på filen → vælg **Copy as path**
4. Gem stien — du skal bruge den i Claude config senere

Eksempel på sti:
- `C:\MCP_HTTP\dotnet\MssqlMcp\bin\Debug\net8.0\MssqlMcp.exe`



---

## 4) Find din connection string

Du skal bruge en connection string til din SQL Server database.

### Eksempler

**Lokal SQL Server (Windows login):**
```text
Server=.;Database=MyDb;Trusted_Connection=True;TrustServerCertificate=True
```

**SQL Express (Windows login):**
```text
Server=.\SQLEXPRESS;Database=MyDb;Trusted_Connection=True;TrustServerCertificate=True
```

**Remote SQL Server (SQL login):**
```text
Server=SERVERNAVN;Database=MyDb;User ID=USERNAME;Password=PASSWORD;Encrypt=True;TrustServerCertificate=True
```

**Azure SQL (SQL login):**
```text
Server=tcp:SERVER.database.windows.net,1433;Database=MyDb;User ID=USERNAME;Password=PASSWORD;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

### Hvor finder man den typisk?
- **Azure SQL:** Azure Portal → SQL Database → *Connection strings* (vælg ADO.NET eller ODBC og tilpas)
- **On-prem SQL:** få server/instance + db-navn + auth-type fra IT/DBA (og byg string ud fra det)

---

## 5) Opsæt Claude Desktop config

Claude Desktop skal have en config-fil, der fortæller den hvordan den starter MCP-serveren.

### 5.1 Find config-filen

1. Tryk **Win + R**
2. Skriv:
   ```text
   %APPDATA%
   ```
3. Tryk **Enter**
4. Åbn mappen **Claude**
5. Åbn filen:
   - `claude_desktop_config.json`

Hvis filen ikke findes:
- Opret en ny fil med navnet **claude_desktop_config.json**

> Typisk sti:  
> `C:\Users\<brugernavn>\AppData\Roaming\Claude\claude_desktop_config.json`

---

## 6) Indsæt MCP-server i config (copy/paste)

Kopiér følgende JSON ind i `claude_desktop_config.json`.

✅ VIGTIGT: Du skal ændre:
- `command` → til din egen sti til `MssqlMcp.exe`
- `CONNECTION_STRING` → til din egen database connection string

```json
{
  "mcpServers": {
    "MSSQL MCP": {
      "command": "C:\\PATH\\TO\\MssqlMcp.exe",
      "env": {
        "CONNECTION_STRING": "Server=localhost;Database=MyDb;User ID=USERNAME;Password=PASSWORD;Encrypt=True;TrustServerCertificate=True",
        "TABLE_SYNONYMS": "{\"SalesStats_Today\": [\"salg\", \"omsætning\", \"dagens salg\"], \"Customers\": [\"kunder\", \"kundeinformation\"], \"Items\": [\"varer\", \"produkter\"], \"Orders\": [\"ordrer\", \"bestillinger\"]}",
        "COLUMN_SYNONYMS": "{\"LineAmountLCY\": [\"bel\\u00f8b\", \"salgsv\\u00e6rdi\", \"oms\\u00e6tning lokal valuta\"], \"LineAmountEUR\": [\"beløb i euro\", \"omsætning i eur\"], \"Company\": [\"land\", \"kunde landekode\"], \"SalesID\": [\"ordre-id\", \"ordrenummer\"]}",
        "SCHEMA_HINTS": "{\"llm\": \"Primært schema for salgs-, kunde- og logistikdata\", \"dbo\": \"Standard schema for generelle stamdata og systemtabeller\"}",
        "CURRENCY_HINTS": "{\"LCY\": \"Local Currency: NO=NOK, SE=SEK, DK=DKK. Præsentér LCY i korrekt lokal valuta pr. land.\", \"EUR\": \"Brug EUR som default til sammenlignelige beløb på tværs af lande. Hvis brugeren spørger pr. land, giv både EUR og LCY.\"}",
        "DEFAULT_TIMEZONE": "Europe/Copenhagen",
        "LOG_LEVEL": "Information"
      }
    }
  }
}
```

### Vigtigt om Windows-stier
I JSON skal backslashes skrives dobbelt:
- ✅ `C:\\Atlytix\\MSSQL-MCP\\MssqlMcp.exe`
- ❌ `C:\Atlytix\MSSQL-MCP\MssqlMcp.exe`

---

## 7) Genstart Claude Desktop

1. Luk Claude Desktop helt
2. Åbn Claude Desktop igen

Claude læser config’en ved opstart.

---

## 8) Test at MCP virker

Åbn en ny chat i Claude og skriv fx:

1) **List tabeller**
> “Brug MCP tools til at liste tabeller i databasen.”

2) **Beskriv tabel**
> “Beskriv tabellen Customers (DescribeTable).”

3) **Læs data**
> “Læs 10 rækker fra Customers (ReadData).”

Hvis du får resultater tilbage, virker opsætningen.

---

## 9) Fejlsøgning (hurtigt)

### Problem: “Tools vises ikke i Claude”
- Tjek at config-filen ligger i `%APPDATA%\Claude\`
- Tjek at JSON er valid (kommaer/klammer)
- Genstart Claude helt

### Problem: SQL connection fejler
- Tjek servernavn/instance er korrekt (fx `.` vs `.\SQLEXPRESS`)
- Test connection string i SSMS/Azure Data Studio
- Prøv midlertidigt `Database=master` for at tjekke adgang
- Hvis remote: tjek firewall/VPN/port (typisk 1433)

---

## 10) Sikkerhed (anbefalinger)

- Undgå at hardcode passwords i repos — brug helst en **lokal config** hos kunden.
- Hvis I skal dele en repo: brug placeholders og bed kunden selv indsætte credentials lokalt.
- Overvej at oprette en dedikeret SQL-bruger med mindst mulige rettigheder (read-only hvor muligt).

---

## Klar-til-kunde tjekliste

- [ ] Zip downloadet
- [ ] Udpakket i valgfri mappe
- [ ] Fundet sti til `MssqlMcp.exe`
- [ ] Fundet database connection string
- [ ] Opdateret `claude_desktop_config.json`
- [ ] Genstartet Claude
- [ ] Testet med ListTables/DescribeTable/ReadData

---

## (Valgfrit) “Retrieve Knowledge først” arbejdsgang

Nogle Claude-versioner understøtter ikke et `initialPrompt`-felt i config. Hvis I vil sikre korrekt flow, så brug denne rutine:

1) Stil spørgsmålet  
2) Kald først `Retrieve Knowledge` (hvis tool findes)  
3) Brug derefter SQL-tools (`ListTables`, `DescribeTable`, `ReadData`)

Hvis I ønsker, kan denne regel også bygges ind i MCP-serveren, så det altid sker automatisk.

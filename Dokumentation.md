# Skapa data
För att skapa data använde vi ChatGPT med följande prompts:
> Hey, could you create 2 datasets in csv. First SalesData with the columns SaleID, ProductName, Category, Quantity, PricePerUnit and SalesTimestamp. It should have at least 1000 records and 300 different products. The second dataset is Products with the columns ProductID, ProductName, Stock and Supplier. It shall contain the same products as SalesData, and there should be at least 300 records and at least 30 different suppliers.

Det resulterade i csv filer med rätt antal data men ProductName och Supplier var slumpade, ex. Product_BxXrh. Därför använde vi två ytterligare prompt för att justera filerna:
> Hey, could you replace the nonsense-names (eg Product_BxXrh) with realistic product names?
> Also update the suppliers with more realistic names

# Import av data
Vi har skapat en databas som heter MongoLab i ett kluster som samtliga gruppmedlemmar har åtkomst till. Där skapade vi Collections med namn Products och SalesData. I MongoDB la vi sedan till csv-filerna i sina respektive Collections. Under importen användes de förslagna datatyperna för samtliga kolumner förutom SalesTimestamp där Date användes istället för String.


# Aggregationer
## Total Sales Value
Använder data från SalesData. Summerar ihop det totala försäljningsvärdet. Multiplicerar alltså quantity med PricePerUnit och summerar ihop det totala försäljningsvärdet i databasen.

## Average Sell Value Per Hour
Använder data från SalesData för att beräkna hur mycket som totalt har sålts och hur många timmar som använts för det, för att sedan beräkna medelvärdet per timme.

## TotalSalesPerCategory
Steg 1 `$group`: 
- Grupperar dokumenten efter `category` med hjälp av värdet i fältet `Category`.
- För varje grupp beräknas:
  - **TotalSold**: Det totala antalet sålda enheter, beräknat med `$sum` på fältet `Quantity`.
  - **TotalRevenue**: Den totala intäkten, veräknat genom att multiplicera `Quantity` med `PricePerUnit` och sen summera resultatet för gruppen.

**Exempel på utdata:**
```json
   { "_id": "Sports", "TotalSold": 921, "TotalRevenue": 212621.85 },
   { "_id": "Books", "TotalSold": 822, "TotalRevenue": 200530.07 },
   { "_id": "Toys", "TotalSold": 1052, "TotalRevenue": 236789.37 },
   { "_id": "Electronics", "TotalSold": 772, "TotalRevenue": 209463.76 },
   { "_id": "Clothing", "TotalSold": 869, "TotalRevenue": 228791.47 },
   { "_id": "Home & Kitchen", "TotalSold": 966, "TotalRevenue": 236441.88 }
```

## TopSellingProducts
Steg 1 `$group`: 
- Grupperar dokumenten efter `ProductName` med hjälp av värdet i feltet `ProductName`.
- För varje grupp beräknas:
  - **TotalSold:** Det totala antalet sålda enheter, beräknat med `$sum` på fältet `Quantity`.

Steg 2 `$sort`: 
- Sorterar dokumenten i fallande ordning baserat på värdet i fältet `TotalSold`.

Steg 3 `$limit`:
- Begränsar resultatet till den 5 övre dokumenten.

**Exempel på utdata:**
```json
  { "_id": "Sci-Fi Book", "TotalSold": 306 },
  { "_id": "Yoga Mat", "TotalSold": 259 },
  { "_id": "Sneakers", "TotalSold": 257 },
  { "_id": "Toaster", "TotalSold": 223 },
  { "_id": "Sweater", "TotalSold": 211 }
```

## SalesByHour
Steg 1 `$addFields`:
- Använder `$dateToParts` för att extrahera delar av datumet från fältet `SalesTimestamp`.
- Skapar ett nytt fält, `Hour`, med endast timdelen från SalesTimestamp. 

Steg 2 `$group`: 
- Grupperar dokumenten efter timmen som extraherades innan vilket anges med `_id: "$Hour.hour"`.
- För varje timme beräknas:
  - **TotalSold:** Det totala antalet sålda enheter under den specifika timmen, beräknat med `$sum` på fältet `Quantity`.

Steg 3 `$match`:
- Filtrerar grupperna så att endast timmar mellan 0 och 23 (inklusive) inkluderas i resultatet.
  
Steg 4 `$sort`:
- Begränsar resultatet till en dast timme `_id: 1`.

**Exempel på utdata:**
```json
  { "_id": 0, "TotalSold": 249 },
  { "_id": 1, "TotalSold": 233 },
  { "_id": 2, "TotalSold": 244 },
  { "_id": 3, "TotalSold": 228 }
```

# Att skapa KPI-Visualiseringar
Vi valde 3 KPI'er baserade på uppgiftens beskrivning. 

Top 5 products:
Stapeldiagram som visar dom 5 bästsäljande produkter. x-axeln visar antal många produkter som säljs och y-axel visar produktnamn.

Total Revenue Per Category: Visar Totala intäkter per kategori från TotalSalesPerCategory.

Cirkeldiagram som visar lagerstatus från Products med Stock och ProductName.

Två filter skapades för att kunna filtrera kategorier där Stapeldiagramet Total Revenue Per Category påverkas samt att kunna välja tidsperiod där linjediagrammet Hourly Sales påverkas.

Hourly Sales Trend: Linjediagram med timme (x-axel) och antal sålda enheter (y-axel). Använde oss av "HourlySales" som datakälla, vars id är vilken timme på dygnet det är och sedn värdet på alla försäljningar den timmen. Efter lagt in rätta X och Y värden så gjorde vi kurvorna lite mjukare samt satte punkter vid timmarna så att det lättare visuellt att förstå. Ändrade titlar och färg. Den kan hjälpa oss förstå när det handlas mest under dagen, med det kan man tillexempel planera in när man ska fylla på varor. 


# Widgets

MVI -Most valued Item - det är ju den produkt som sålt bäst. Den använder sig av samma vy som top 5 produkter, men är bara en kul grej för att väldigt snabbt och väldigt enkelt kunna se vad som säljer bäst 

Total sales value - För att se värdet av allt sålt.

Average sell by hour - För att se en sammanfattning för den genomsnittliga försäljning varje timme.

# Vilka är fördelarna med att använda MongoDB för detta ändamål?

MongoDB är det mest populära NoSQL-databasen.  Den är lätt att lära sig och har enkla redskap för att få en bra övergripande information visuellt (genom att använda atlasDB). Den har öppenkällkod, men kan också användas av större företag i en liten annan version. Den går att använda lokalt men också i moln. Det är en dokumentbaserad databas. 

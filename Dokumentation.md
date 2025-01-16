# Skapa data
För att skapa data använde vi ChatGPT med följande prompts:
> Hey, could you create 2 datasets in csv. First SalesData with the columns SaleID, ProductName, Category, Quantity, PricePerUnit and SalesTimestamp. It should have at least 1000 records and 300 different products. The second dataset is Products with the columns ProductID, ProductName, Stock and Supplier. It shall contain the same products as SalesData, and there should be at least 300 records and at least 30 different suppliers.

Det resulterade i csv filer med rätt antal data men ProductName och Supplier var slumpade, ex. Product_BxXrh. Därför använde vi två ytterligare prompt för att justera filerna:
> Hey, could you replace the nonsense-names (eg Product_BxXrh) with realistic product names?
> Also update the suppliers with more realistic names

# Import av data
Vi har skapat en databas som heter MongoLab i ett kluster som samtliga gruppmedlemmar har åtkomst till. Där skapade vi Collections med namn Products och SalesData. I MongoDB la vi sedan till csv-filerna i sina respektive Collections. Under importen användes de förslagna datatyperna för samtliga kolumner förutom SalesTimestamp där Date användes istället för String.


#Fortsätt här :)


# Att skapa KPI-Visualiseringar


Hourly Sales Trend: Linjediagram med timme (x-axel) och antal sålda enheter (y-axel). Använde oss av "HourlySales" som datakälla, vars id är vilken timme på dygnet det är och sedn värdet på alla försäljningar den timmen. Efter lagt in rätta X och Y värden så gjorde vi kurvorna lite mjukare samt satte punkter vid timmarna så att det lättare visuellt att förstå. Ändrade titlar och färg. Den kan hjälpa oss förstå när det handlas mest under dagen, med det kan man tillexempel planera in när man ska fylla på varor. 


# Widgets

MVI -Most valued Item - det är ju den produkt som sålt bäst. Den använder sig av samma vy som top 5 produkter, men är bara en kul grej för att väldigt snabbt och väldigt enkelt kunna se vad som säljer bäst 

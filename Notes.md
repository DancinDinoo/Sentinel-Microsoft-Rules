A Guide to KQL 

Syntax -  

Just to start with its good to understand the Syntax KQL uses, there’s little niche stuff that comes with using different functions but generally you want to keep an eye out for 
If you’re declaring any new variable, so say you’re starting out a query like below, you need to put a semi colon after the end of the statement otherwise it’ll throw up an error. These need to be the FIRST thing we do if we’re writing queries  
Let User = Dino@TRex.com; 

We use “ “ to specify strings we’re searching for 
| where Username == “User” 

Notice the difference between the variable = and the Boolean ==, this is because one is setting a variable to = something, the where statement is acting like a yes or a no answer, we’re saying where Username IS equal to our example User. 

Also we gotta use | before every operator, it’s called piping like you’re piping the data you’ve got & doing something else with it, so you’re operating on it.  

You can put ! before = to say something doesn’t equal rather than equal 

You can us * as what’s called a wildcard, so if you aren’t sure what column has the information you need or think multiple might you can do something like this: 

| where * == dsfhdsogh 

This will return any matches on that value regardless of what column its under. 

Cool thing I learnt, if you right click on a variable when you’ve searched something you can exclude/include it in a query, which might help when you’ve got spaces/stuff inside a Json string 

Lists -  

When we’re editing rules inside of Sentinel its useful to implement allow lists so that we can streamline false positives, to do this we need to create a list of whatever it is, IPs, Hosts etc so we can update it easily. 
Let AllowList = dynamic([“User1”, “User2”, “User3”]) 
SecurityAlert 
| where Usernames !in (AllowList) 
Okay so we used a dynamic array as its convenient & accepts all data types.  
We’re querying the SecurityAlert table where all the data is stored that we want to query. 
Then we’re using the where operator to look for something, but here because we’re using ! beforehand we’re saying where the Usernames ARE NOT in the allow list we created.  

To create those allowlists you can also use
pack_array = ("Whatever");
This also packs them into a dynamic array... Slightly unsure what the difference between that & dynamic is but I'm sure I'll figure that out

let examplelist = dynamic(["UK", "US", "FR"]); 
union SigninLogs, SecurityAlert 
| where Location in(examplelist) 
So if you paste that into the Logs search you should see some results, we’re basically searching for any results where Location is = to the country codes seen above. 
If you want to search multiple tables for data, you can use the union operator to search multiple tables at once. 

Another type of List you can use is the make_set, this involves taking a collection of individual values in a column and passing them into a Json Array that you can then manipulate as you wish.  
SigninLogs 
| summarize count (), 
Users = make_set(AlternateSignInName) by Location 
| project Users, Location 
So if you run this query what it’ll show all the users inside of a list who have accessed via a location, if you add more variables after the by this will split this up even further. 

So if you’ve got a set you can use the following to identify how many unique values are inside that set 
VMConnection 
| summarize count(), 
Ports = make_set(DestinationPort) by SourceIp 
| project NumberofPorts = array_length(Ports), SourceIp, Ports 
This will create a set of ports that have been hit by a specific source IP and put this into a Json list, then we call the list into the array_length function in Project naming this NumberOfPorts which will give us a value of how many unique ports are inside that list. 

Displaying Data – 

Once you’ve slimmed your data down using where/not etc you need to then do something with it, if you don’t use specify what you want it will just show you all the columns available, which can be overwhelming.
Let User = Dino@TRex.com
Security Alert 
| where AlternateSignInName == User 
This will show all the columns available where User = AlternativeSignInName 

By adding the below
| project Username = AlternateSignInName, Location 
the query does the same thing, but instead of returning any columns associated with that query it will only the AlternateSignInName which I have renamed Username & Location. If you want to show another column of information just add a comma & repeat as above. 

You can use summarize to return an aggregational function of a column, so you’re doing some maths usually to calculate a value, most commonly you’ll be using either count or a variation of it but there’s loads of others you can use. 
let user = "Dino@TRex.com"; 
Signinlogs
| summarize hits = count () by user 
| project hits, user 
So here we’re counting the number of times we have seen the user in a period of time & then projecting that information. You can further expand this count by adding more variables after the by. 

let user = "Dino@TRex.com"; 
SigninLogs
| where AlternateSignInName == user  
| summarize hits = count () by user, Location 
| project hits, user
| sort by Hits desc
So now we’ve added location to the count, it will count how many instances the user & location have the same value pair, if one instance was user GB and the other user FR these would be counted separately. 
Adding the sort at the end lets you sort the column how you like, usually its asc/desc depending on what you're after

You can use dcount with will return the number of unique values are in that column 
SigninLogs
| summarize dcount(user) 

Different to dscount, distinct will show you all the unique values in the column you're searching as a new table
SigninLogs
| distinct UserDisplayName

Manipulating Data – 

Because Azure has so many different tables/columns to search through it can be convenient to combine some of that data into a single column searching or evaluating 
union VMConnection, AzureNetworkAnalytics_CL 
| extend SourceAddress = strcat(SrcIP_s, SourceIp) 
So using the extend function we’ve created a new column called SourceAddress and merged the values in both SrcIP_s & SourceIp using atrcat. This might be useful for the counting or querying further. 

So if you have information that comes in as an XML string such as the SecurityEvent/Sysmon fields, you need to convert it into a Json to be able to manipulate it easily which you can do with 
| extend EVData = parse_xml(EventData) 
| extend EventDetails = EVData.EventData 
So we’re creating a new column called EVData, our XML Data is called EventData so we’re using the parse_xml function on this. Next we’re creating a new column for our Json value called EventDetails, and calling our column of parsed XML Data EVData. 
Depending on how many breaks your XML data has, so that’s any of these <\Thing> <\EventData> is how many . your statement needs to have. So in this case it would be EVData.Thing.EventData. 

| extend split(“STRING”/VARIABLE “CHARACTER”) [NUMBER] 
If you want to grab anything after a certain point in the string and then display this as a separate column, you can use this to grab anything after the CHARACTER after it has occurred NUMBER times 



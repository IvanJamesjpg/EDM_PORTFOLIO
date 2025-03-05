#MIDTERM TASK 2

let
    Source = Csv.Document(File.Contents("D:\I101\Uncleaned_DS_jobs.csv"),[Delimiter=",", Columns=15, Encoding=1252, QuoteStyle=QuoteStyle.Csv]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"index", Int64.Type}, {"Job Title", type text}, {"Salary Estimate", type text}, {"Job Description", type text}, {"Rating", type number}, {"Company Name", type text}, {"Location", type text}, {"Headquarters", type text}, {"Size", type text}, {"Founded", Int64.Type}, {"Type of ownership", type text}, {"Industry", type text}, {"Sector", type text}, {"Revenue", type text}, {"Competitors", type text}}),
    #"Extracted Text Before Delimiter" = Table.TransformColumns(#"Changed Type", {{"Salary Estimate", each Text.BeforeDelimiter(_, "("), type text}}),
    #"Inserted Text Between Delimiters" = Table.AddColumn(#"Extracted Text Before Delimiter", "Min Sal", each Text.BetweenDelimiters([Salary Estimate], "$", "-"), type text),
    #"Inserted Text Between Delimiters1" = Table.AddColumn(#"Inserted Text Between Delimiters", "MAX Sal", each Text.BetweenDelimiters([Salary Estimate], "$", " ", 1, 0), type text),
    #"Added Custom" = Table.AddColumn(#"Inserted Text Between Delimiters1", "Role Type", each if Text.Contains([Job Title], "Data Scientist") then
"Data Scientist"
else if Text.Contains([Job Title], "Data Analyst") then
"Data Analyst"

else if Text.Contains([Job Title], "Data Engineer") then
"Data Engineer"
else if Text.Contains([Job Title], "Machine Learning") then
"Machine Learning Engineer"
else
"other"),
    #"Changed Type1" = Table.TransformColumnTypes(#"Added Custom",{{"Role Type", type text}}),
    #"Added Custom1" = Table.AddColumn(#"Changed Type1", "Custom", each if [Location]= "New Jersey" then ", NJ"
else if [Location] = "Remote" then ", other"
else if [Location]= "United States" then ", other"
else if [Location]= "Texas" then ", TX"
else if [Location]= "Patuxent" then ", MA"
else if [Location]= "California" then ", CA"
else if [Location]= "Utah" then ", UT"
else [Location]),
    #"Renamed Columns" = Table.RenameColumns(#"Added Custom1",{{"Custom", "Location Correction"}}),
    #"Split Column by Delimiter" = Table.SplitColumn(#"Renamed Columns", "Location Correction", Splitter.SplitTextByDelimiter(",", QuoteStyle.Csv), {"Location Correction.1", "Location Correction.2"}),
    #"Changed Type2" = Table.TransformColumnTypes(#"Split Column by Delimiter",{{"Location Correction.1", type text}, {"Location Correction.2", type text}}),
    #"Replaced Value" = Table.ReplaceValue(#"Changed Type2","Anne Rundell","MA",Replacer.ReplaceText,{"Location Correction.2"}),
    #"Renamed Columns1" = Table.RenameColumns(#"Replaced Value",{{"Location Correction.2", "State Abbreviations"}}),
    #"Split Column by Delimiter1" = Table.SplitColumn(#"Renamed Columns1", "Size", Splitter.SplitTextByDelimiter("to", QuoteStyle.Csv), {"Size.1", "Size.2"}),
    #"Changed Type3" = Table.TransformColumnTypes(#"Split Column by Delimiter1",{{"Size.1", type text}, {"Size.2", type text}}),
    #"Replaced Value1" = Table.ReplaceValue(#"Changed Type3","employees","",Replacer.ReplaceText,{"Size.1", "Size.2"}),
    #"Inserted Number" = Table.AddColumn(#"Replaced Value1", "MinCompanySize", each Number.From([Size.1]), type number),
    #"Inserted Number1" = Table.AddColumn(#"Inserted Number", "MaxCompanySize", each Number.From([Size.2]), type number),
    #"Replaced Value2" = Table.ReplaceValue(#"Inserted Number1","-1","N/A",Replacer.ReplaceText,{"Competitors"}),
    #"Replaced Value3" = Table.ReplaceValue(#"Replaced Value2","negative","0",Replacer.ReplaceText,{"Revenue"}),
    #"Replaced Value4" = Table.ReplaceValue(#"Replaced Value3","-1","other",Replacer.ReplaceText,{"Industry"}),
    #"Removed Columns" = Table.RemoveColumns(#"Replaced Value4",{"Size.1", "Size.2"}),
    #"Split Column by Delimiter2" = Table.SplitColumn(#"Removed Columns", "Company Name", Splitter.SplitTextByDelimiter("#(lf)", QuoteStyle.Csv), {"Company Name.1", "Company Name.2"}),
    #"Changed Type4" = Table.TransformColumnTypes(#"Split Column by Delimiter2",{{"Company Name.1", type text}, {"Company Name.2", type number}}),
    #"Removed Columns1" = Table.RemoveColumns(#"Changed Type4",{"Company Name.2", "Job Description"})
in
    #"Removed Columns1"

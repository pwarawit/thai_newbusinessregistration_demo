# thai_newbusinessregistration_demo
This demo how to analyze Thailand New Business Registration data. The objective is to document all the required steps and consideration 

## Source Data:
http://www.dbd.go.th/ewt_news.php?nid=12385&filename=index

### Download each of the CSV files into the directory (this github repo also contain the data)

### Directory Structure
-- data_original : where we keep all the csv files downloaded from source

สาธิตการวิเคราะห์ข้อมูล การจดทะเบียนธุรกิจใหม่
 ## แหล่งข้อมูล
 แหล่งข้อมูลของโครงการนี้ ได้มาจากหน่วยงานราชการ

นอกจากตัวข้อมูลเองแล้ว ยังต้องใช้ทำ data transformation & mapping additional data ด้วย 
http://www.dbd.go.th/download/PDF_/book_business_man.pdf ข้อมูลเสริมเพื่อสร้าง master data hierarchy 
 
ต้องมีการสร้างไฟล์ Type Level ขึ้นอีก 5 ไฟล์เพื่อเก็บข้อมูล Business Type Hierarchy Level อีก 5 ระดับ

มาดูขั้นตอนการ merge file กันก่อน
<pre>
let
    Source = Csv.Document(File.Contents("C:\Users\PanaEk\Documents\GitHub\thai_newbusinessregistration_demo\data_original\99_201605.csv"),[Delimiter=",", Columns=8, Encoding=874, QuoteStyle=QuoteStyle.Csv]),
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Column1", type text}, {"Column2", type text}, {"Column3", type text}, {"Column4", type text}, {"Column5", type text}, {"Column6", type text}, {"Column7", type text}, {"Column8", type text}}),
    #"Changed Type with Locale" = Table.TransformColumnTypes(#"Changed Type", {{"Column4", type date}}, "th-TH"),
    #"Removed Columns" = Table.RemoveColumns(#"Changed Type with Locale",{"Column1"}),
    #"Renamed Columns" = Table.RenameColumns(#"Removed Columns",{{"Column2", "TaxID"}, {"Column3", "Name"}, {"Column4", "Registered Date"}}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Renamed Columns",{{"Column5", Int64.Type}}),
    #"Renamed Columns1" = Table.RenameColumns(#"Changed Type1",{{"Column5", "Capital"}, {"Registered Date", "Date"}, {"Column6", "Type Code"}, {"Column7", "Objective Detail"}, {"Column8", "Address"}}),
    #"Removed Top Rows" = Table.Skip(#"Renamed Columns1",3)
in
    #"Removed Top Rows"
</pre>
	
แล้วทำการสร้าง duplicate table ตั้งชื่อใหม่เป็น New Registrations

TSIC 2552 เป็นรหัสธุรกิจ สามารถดาวน์โหลดได้จาก http://www.dbd.go.th/download/doc/table_TSIC2552.xls
แต่ไฟล์นี้มีแต่ระดับ lowest level กับ mapping ขึ้นไปกับแต่ละระดับ จำเป็นต้องมี level 1-4 ต่างหากอีก
 
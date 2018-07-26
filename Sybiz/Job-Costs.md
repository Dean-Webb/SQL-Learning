How to look up job costs
```
select "jc.Job"."JobId",
	"jc.Job"."Code", "jc.Job"."Description",
       "jc.Job"."ExtendedDescription",
       "dr.Customer"."Name",
       "dr.Customer"."Code" as "dr.Customer_Code",
       "jc.JobGroup"."Description" as "JobGroup",
       "jc.Job"."ContractValue",
       CONVERT(VARCHAR(10),"jc.Job"."StartDate",103) as StartDate,
	CONVERT(VARCHAR(10),"jc.Job"."TargetCompletionDate",103) as TargerCompletion,
            "cm.Status"."Description" as "Status",
	"jc.JobExtendedProperty"."Total Quote",
       "jc.JobExtendedProperty"."Is Packing/Freight charged sep",
       "jc.JobExtendedProperty"."Freight",
       "jc.JobExtendedProperty"."Equipment",
       "jc.JobExtendedProperty"."Ancilaries",
       "jc.JobExtendedProperty"."Packaging",
	"cm.SortCode"."Description" as "Sort Detail",
       "cm.SortCode"."Code" as "Sort Code",
	IIF("jc.JobExtendedProperty"."Is Packing/Freight charged sep" = 0,("jc.JobExtendedProperty"."Freight" + "jc.JobExtendedProperty"."Equipment" + "jc.JobExtendedProperty"."Ancilaries" + "jc.JobExtendedProperty"."Packaging"),("jc.JobExtendedProperty"."Equipment" + "jc.JobExtendedProperty"."Ancilaries"))AS "Calculated Job Value",
	IIF("cm.SortCode"."Code" = 'NR',0,IIF("cm.SortCode"."Code" = 'SR',"jc.JobExtendedProperty"."Equipment" * 0.075,IIF("cm.SortCode"."Code" = 'RR' AND "jc.JobGroup"."Description" = 'Rotex Machines',"jc.JobExtendedProperty"."Equipment" * 0.0375,"jc.JobExtendedProperty"."Equipment" * 0.05))) as Royalty,
	IIF((JobInv.InvSum_Total - JobInv.InvSum_Tax)<>0,(JobInv.InvSum_Total - JobInv.InvSum_Tax),0) AS "InvoiceOnly",
	IIF((-(JobCrd.CrdSum_Total - JobCrd.CrdSum_Tax))<>0,(-(JobCrd.CrdSum_Total - JobCrd.CrdSum_Tax)),0) AS "Credits",
	(IIF((JobInv.InvSum_Total - JobInv.InvSum_Tax)<>0,(JobInv.InvSum_Total - JobInv.InvSum_Tax),0))+(IIF((-(JobCrd.CrdSum_Total - JobCrd.CrdSum_Tax))<>0,(-(JobCrd.CrdSum_Total - JobCrd.CrdSum_Tax)),0))  As "Invoiced",
	("jc.Job"."ContractValue"-((IIF((JobInv.InvSum_Total - JobInv.InvSum_Tax)<>0,(JobInv.InvSum_Total - JobInv.InvSum_Tax),0)) + (IIF((-(JobCrd.CrdSum_Total - JobCrd.CrdSum_Tax))<>0,(-(JobCrd.CrdSum_Total - JobCrd.CrdSum_Tax)),0)))) As "Order Book",
	(IIF(RoySuppInv.SuppRoyInv<>0,RoySuppInv.SuppRoyInv,0)-IIF(RoySuppCrd.SuppRoyCrd<>0,RoySuppCrd.SuppRoyCrd,0)) AS "Supplier Royalty Inv Crd",
	IIF(RoySuppCrd.SuppRoyCrd<>0,RoySuppCrd.SuppRoyCrd,0) AS "Supplier Royalty Crd",
	(IIF("cm.SortCode"."Code" = 'NR',0,(IIF("cm.SortCode"."Code" = 'SR',"jc.JobExtendedProperty"."Equipment" * 0.075,IIF("cm.SortCode"."Code" = 'RR' AND "jc.JobGroup"."Description" = 'Rotex Machines',"jc.JobExtendedProperty"."Equipment" * 0.0375,"jc.JobExtendedProperty"."Equipment" * 0.05))-(IIF(RoySuppInv.SuppRoyInv<>0,RoySuppInv.SuppRoyInv,0)-IIF(RoySuppCrd.SuppRoyCrd<>0,RoySuppCrd.SuppRoyCrd,0))))) as "Royalty to apply",
	((IIf(CJ.SumJobCost<>0,CJ.SumJobCost,0))-(IIf(IJ.CostInvoiced<>0,IJ.CostInvoiced,0))) AS "WIP"


  from (((((((((("jc"."Job" "jc.Job"
 	full join (
	 	Select 
		"jc.JobCost"."JobId",
		sum("jc.JobCost"."ForeignTotal") AS TotalInvoiced
		from "jc"."JobCost" "jc.JobCost"
		GROUP BY "jc.JobCost"."JobId")CostJob 
       on (CostJob."JobId" = "jc.Job"."JobId"))
  left join "jc"."JobGroup" "jc.JobGroup"
       on ("jc.JobGroup"."JobGroupId" = "jc.Job"."JobGroupId"))
  Left join "dr"."Customer" "dr.Customer"
       on ("dr.Customer"."CustomerId" = "jc.Job"."CustomerId"))
  left join "cm"."Status" "cm.Status"
       on ("cm.Status"."StatusId" = "jc.Job"."StatusId"))
inner join "jc"."JobExtendedProperty"
       "jc.JobExtendedProperty"
       on ("jc.JobExtendedProperty"."ObjectId" = "jc.Job"."JobId"))
 inner join "cm"."SortCode" "cm.SortCode"
       on ("cm.SortCode"."SortCodeId" = "jc.Job"."SortCodeId"))
	Left Join (
	   	select 
			sum("dr.SalesInvoiceLine"."Tax") as "InvSum_Tax",
     			sum("dr.SalesInvoiceLine"."Total") as "InvSum_Total",
			"dr.SalesInvoiceLine"."JobId"
			
 			from ("jc"."Job" "jc.Job"
			inner join "dr"."SalesInvoiceLine"
      		"dr.SalesInvoiceLine"
       		on ("dr.SalesInvoiceLine"."JobId" = "jc.Job"."JobId"))
			group by "dr.SalesInvoiceLine"."JobId"	   
	   		) JobInv on  (JobInv."JobId" ="jc.Job"."JobId"))
	Left Join (
	   	select 
			sum("dr.SalesCreditLine"."Tax") as "CrdSum_Tax",
     			sum("dr.SalesCreditLine"."Total") as "CrdSum_Total",
			"dr.SalesCreditLine"."JobId"
			
 			from ("jc"."Job" "jc.Job"
			inner join "dr"."SalesCreditLine"
      		"dr.SalesCreditLine"
       		on ("dr.SalesCreditLine"."JobId" = "jc.Job"."JobId"))
			group by "dr.SalesCreditLine"."JobId"	   
	   		) JobCrd on  (JobCrd."JobId" ="jc.Job"."JobId"))
Left Join(
	select 
		"cr.SupplierGroup"."Description" as "Supplier Group",
       	"cr.Supplier"."Name",
       	sum("cr.PurchaseInvoiceLine"."ForeignTotal") as "SuppRoyInv",
       	"jc.Job"."JobId"
  
  		from (((("cr"."SupplierGroup" "cr.SupplierGroup"
  		inner join "cr"."Supplier" "cr.Supplier"
       on ("cr.Supplier"."GroupId" = "cr.SupplierGroup"."SupplierGroupId"))
  		inner join "cr"."PurchaseInvoice"
       "cr.PurchaseInvoice"
       on ("cr.PurchaseInvoice"."SupplierId" = "cr.Supplier"."SupplierId"))
  		inner join "cr"."PurchaseInvoiceLine"
       "cr.PurchaseInvoiceLine"
       on ("cr.PurchaseInvoiceLine"."PurchaseInvoiceId" = "cr.PurchaseInvoice"."PurchaseInvoiceId"))
  		inner join "jc"."Job" "jc.Job"
       on ("jc.Job"."JobId" = "cr.PurchaseInvoiceLine"."JobId"))
		 where
       ("cr.SupplierGroup"."Description" = N'Royalty')
		group by "jc.Job"."JobId", "cr.Supplier"."Name",
       "cr.SupplierGroup"."Description"
       ) RoySuppInv on (RoySuppInv."JobId" = "jc.Job"."JobId"))



Left Join(     
       select "cr.SupplierGroup"."Description" as "Supplier Group",
       "cr.Supplier"."Name",
       sum("cr.PurchaseCreditLine"."ForeignTotal") as "SuppRoyCrd",
       "jc.Job"."JobId"
  from (((("cr"."SupplierGroup" "cr.SupplierGroup"
  inner join "cr"."Supplier" "cr.Supplier"
       on ("cr.Supplier"."GroupId" = "cr.SupplierGroup"."SupplierGroupId"))
  inner join "cr"."PurchaseCredit"
       "cr.PurchaseCredit"
       on ("cr.PurchaseCredit"."SupplierId" = "cr.Supplier"."SupplierId"))
  inner join "cr"."PurchaseCreditLine"
       "cr.PurchaseCreditLine"
       on ("cr.PurchaseCreditLine"."PurchaseCreditId" = "cr.PurchaseCredit"."PurchaseCreditId"))
  inner join "jc"."Job" "jc.Job"
       on ("jc.Job"."JobId" = "cr.PurchaseCreditLine"."JobId"))
 where
       ("cr.SupplierGroup"."Description" = N'Royalty')
group by "cr.Supplier"."Name",
       "cr.SupplierGroup"."Description",
       "jc.Job"."JobId") RoySuppCrd on (RoySuppCrd."JobId" = "jc.Job"."JobId"))





 Left Join 
		(
	select "trn.PostJC"."JobId" AS CJobId,
       	sum(("trn.PostJC"."Debit")-("trn.PostJC"."Credit")) as "SumJobCost"      
 	from  "trn"."PostJC" "trn.PostJC"
   		group by "trn.PostJC"."JobId"
) CJ
	    on   "jc.Job"."JobId" = CJ."CJobId"



	Left Join
	(
	select "trn.PostJI"."JobId" AS IJobId,
       	sum("trn.PostJI"."Cost") as "CostInvoiced"      
 	from  "trn"."PostJI" "trn.PostJI"
   		group by "trn.PostJI"."JobId"
) IJ
	    on   "jc.Job"."JobId" = IJ."IJobId"
		
		
SELECT @Id AS Id, @UserId AS UserId, @SystemDate AS SystemDate
```

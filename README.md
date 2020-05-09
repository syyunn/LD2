# LD2
All information related to the LD2

Currently our `consolidated_layer_reports.reports` represent the raw data, `LD-2`.

- `LD-2` is available at [here](https://disclosurespreview.house.gov/?index=%22lobbying-disclosures%22&size=10&filters={%22reportYear%22:[%222012%22]}&sort=[{%22_score%22:true},{%22field%22:%22registrant.name%22,%22order%22:%22asc%22}]) by clicking `Lobbying Disclosure`
- `LD-2`'s detailed filing instruction is available at [here](https://lda.congress.gov/LD/help/default.htm)

Our current table - `consolidated_layer_reports.reports` has a attribute column, `amount`, which comes from the `LD-2`'s `INCOME` or `EXPENSE`, which are correspondent to `Line12` and `Line13` of the form respectively. (Check [here](https://disclosurespreview.house.gov/ld/ldxmlrelease/2012/Q4/300540389.xml) for the sample `LD-2`)

 
According to the [official instruction](https://lda.congress.gov/LD/help/default.htm), when filing `LD-2`,
- One must complete the income summary if lobbying on behalf of a client
- One must complete the expense summary if you are lobbying on your own behalf

Therefore, if we know whether the `amount` attribute value comes from `INCOME` or `EXPENSE`, we could figure out whether the `_client_uuid` points the same entity with the `_reigstrant_uuid` or not.

For example, let's see the below example `LD-2`:
![Screen Shot 2020-05-08 at 1 22 48 AM](https://user-images.githubusercontent.com/21968222/81319306-72adae80-90ca-11ea-8aa6-3198fd72e09b.png)

This `LD-2` is registered by `Ally Financials Inc.` and its client is itself, `Ally Financials Inc`.
We guarantee that those two are the same and unique entity by the information `EXPENSE` field filed, however, at the `db level`, we could only partially assume they are same entity based on the same name since we have different `_uuid` for client and registrants, and we don't have relation between them.

```sql
select c."_client_uuid", c."client_full_name", r."_registrant_uuid", regist."registrant_full_name"
from consolidated_layer_reports.reports r 
	inner join consolidated_layer_reports.clients as c using ("_client_uuid" )
	inner join consolidated_layer_reports.registrants as regist using ("_registrant_uuid")
where c.client_full_name ILIKE '%Ally Financial%' and regist.registrant_full_name ilike '%Ally Financial%'

>>> _client_uuid	client_full_name	_registrant_uuid	registrant_full_name
>>>c70362bc-70ce-5e00-bef9-cd52d1b1674b	ALLY FINANCIAL INC 	f2460a41-ec8b-5d7d-99a7-c8a37d597d20	ALLY FINANCIAL INC.
```

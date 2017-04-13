If you create an agency fee and the from date is in the past the automatic orp creation will not take affect.  To get this to be run in you must do the following.

### Update any previous successfully runs to allow you to retrigger the process.
``` sql
UPDATE offers
   SET fee_applied = 'N'; -- reset any previous runs.
UPDATE offers
   SET fee_applied = 'Y' 
 WHERE offers.job_id IN (SELECT job_id FROM job_order WHERE job_type_id = 1) ;
--Removes perm roles from process
```
* The update above marks the offers for permenant jobs with y as already updated, because these interfere with the processing of the agency fees.

### Then run the following query that will tell the number of offers that need to be updated:
``` sql
SELECT COUNT (*)
  FROM offers
 WHERE fee_applied = 'N';
```

* Make note of that number. 21699
* (If the updater times out, or something goes wrong, a number of offers will certainly have been updated, and you can confirm that by running the SELECT statement again, and run the updater again to continue.)

### Close agency fee off and insert new fee into database.
* update database to close off and insert new fee
``` sql
SELECT *
  FROM agency_fees
 WHERE fee_type_id = 2;

UPDATE agency_fees
   SET applied_to = trunc(sysdate) - 1
 WHERE id = :agencyFeeId;

INSERT INTO agency_fees (id, fee_type_id, fee_as, amount, applied_from, discipline_unit_id)
VALUES (agency_fees_id_seq.nextval, 2, 'PERCENTAGE', 0.17, TRUNC(SYSDATE), 1);
```

* Update the url below to relate to the new fee 
``` sql
-- Get the agency fee to trigger the run below
SELECT * 
  FROM agency_fees
  JOIN agency_fee_types ON agency_fee_types.ID = agency_fees.fee_type_id
 WHERE fee_type_id = 2 -- procurement
 ORDER BY agency_fees.id desc;
```  

Login into system as admin and then replace url with the following.
http://temps.hsbc.qat.skillstream.net/ssplus/s3/agencyFees/run/41

* remove the number at the end of the above url and replace with the new inserted value.

### Run this statement to check all fees have been run in:

* Check db
``` sql
SELECT count(*)
  FROM offers 
 WHERE status IN (SELECT status_id
                    FROM status_lists
                   WHERE list_name = 'offer_confirmation_and_current')
   AND fee_applied = 'N';
```

* (If there are results then these haven't successfully been processed. If there are remaining offers then hit the url again to run the remaining candidates.)

* Once complete a blank agency fees page will load with a success message.

### Update any successfully runs to allow the auto process to trigger
``` sql
UPDATE offers
   SET fee_applied = 'N'; -- reset any previous runs.
UPDATE offers
   SET fee_applied = 'Y' 
 WHERE offers.job_id IN (SELECT job_id FROM job_order WHERE job_type_id = 1) ;
--Removes perm roles from process
```
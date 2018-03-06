### Finding out the date of the snap shot id
```sql
select * from snap_shot where snap_shot_id = 3200;
```
### Finding the type of the snap ( 6 = Invoice )

```sql
select * from snap_type;
```

### Detailed Time Worked Report - WESTPAC-113 (or 91)

```sql
SELECT job_types.type_desc as job_type,
       job_types.job_type_id,
       cv.first_name,
       cv.last_name,
       offers.internal_id AS employee_id,
       job_order.job_id,
       job_order.job_title,
       managers.first_name AS hm_forename,
       managers.last_name AS hm_surname,
       client_locations.location_desc,
       timesheet_layer.week_end,
       to_char(timesheet_shifts.start_date, 'DAY dd/MM/yyyy') AS day_date,
       timesheet_shifts.start_date,
       timesheet_shifts.end_date,
       timesheet_shifts.break_millis/3600000 break_hours,
       timesheet_shifts.millis/3600000 as shift_hours,
       timesheet_days.days,
       leave_reason.reason_name AS leave_reason,
       timesheet_leave_view.amount as leave_amount,
       timesheet_leave_view.taken_millis AS leave_millis,
       agencies.agency_name,
       CASE
         WHEN timesheet_layer.fully_authorised IS NOT NULL THEN 'authorised'
         WHEN timesheet_layer.unsubmitted_at IS NOT NULL AND timesheet_layer.unsubmitted_at > timesheet_layer.submitted THEN 'unsubmitted'
         WHEN denied_timesheet_layers.denied_date IS NOT NULL THEN 'denied'
         WHEN timesheet_layer.submitted IS NOT NULL THEN 'unauthorised'
       END AS status,
       timesheet_shifts.notes,
       lob3.name AS lob3,
       lob4.name AS lob4,
       lob5.name AS lob5,
       lob6.name AS lob6,
       reports_to_manager.first_name reports_to_manager_first_name,
       reports_to_manager.last_name reports_to_manager_last_name,
       reports_to_manager.employee_id reports_to_manager_employee_id,
       offer_exceptions.text_field_value AS offer_exception
  FROM (SELECT ts_lyr_id, week_end, submitted, unsubmitted_at, fully_authorised, status, job_id, cv_id FROM timesheet_layer) timesheet_layer
  LEFT JOIN (SELECT ts_lyr_id, start_date, end_date, break_millis, millis, notes, 'shift' AS type
             FROM timesheet_shift
             UNION ALL
             SELECT ts_lyr_id, taken_on AS start_date, taken_on AS end_date, 0 AS break_millis, taken_millis AS millis, NULL AS notes, 'leave' as TYPE
             FROM timesheet_leave_view
             UNION ALL
             SELECT ts_lyr_id, labour_date AS start_date, labour_date AS end_date, 0 AS break_millis, NULL AS millis, NULL AS notes, 'days' AS type
             FROM timesheet_days) timesheet_shifts ON timesheet_shifts.ts_lyr_id = timesheet_layer.ts_lyr_id
  LEFT JOIN timesheet_leave_view ON timesheet_leave_view.ts_lyr_id = timesheet_shifts.ts_lyr_id AND timesheet_leave_view.taken_on = timesheet_shifts.start_date AND timesheet_shifts.type = 'leave'
  LEFT JOIN leave_reason ON leave_reason.reason_id = timesheet_leave_view.reason_id
  LEFT JOIN timesheet_days ON timesheet_days.ts_lyr_id = timesheet_shifts.ts_lyr_id AND timesheet_days.labour_date = timesheet_shifts.start_date AND timesheet_shifts.type = 'days'
  LEFT JOIN (SELECT offer_id, cv_id, job_id, agency_id, internal_id, secondary_cost_code FROM offers) offers ON offers.cv_id = timesheet_layer.cv_id AND offers.job_id = timesheet_layer.job_id
  LEFT JOIN (SELECT cv_id, first_name, last_name FROM cv) cv ON cv.cv_id = offers.cv_id
  LEFT JOIN (SELECT job_id, job_title, man_id, location_id, geo_unit_id, job_type_id FROM job_order) job_order ON job_order.job_id = offers.job_id
  LEFT JOIN managers ON managers.man_id = job_order.man_id
  LEFT JOIN client_locations ON client_locations.location_id = job_order.location_id
  LEFT JOIN job_types ON job_types.job_type_id = job_order.job_type_id
  LEFT JOIN LOB6 ON LOB6.leaf_code = offers.secondary_cost_code
  LEFT JOIN LOB5 ON LOB5.leaf_code = offers.secondary_cost_code
  LEFT JOIN LOB4 ON LOB4.leaf_code = offers.secondary_cost_code
  LEFT JOIN LOB3 ON LOB3.leaf_code = offers.secondary_cost_code
  LEFT JOIN (SELECT geo_units_descendants.descendant_geo_unit_id AS geo_unit_id,
                    countries.unit_id as country_id,
                    countries.name
               FROM (SELECT * FROM geo_unit WHERE is_country = 'Y') countries
               JOIN geo_units_descendants ON geo_units_descendants.geo_unit_id = countries.unit_id) countries on countries.geo_unit_id = job_order.geo_unit_id
  LEFT JOIN agencies ON agencies.agency_id = offers.agency_id
  LEFT JOIN (SELECT managers.first_name,
                    managers.last_name,
                    reports_to_manager_offers.offer_id,
                    reports_to_manager_offers.employee_id,
                    managers.man_id
               FROM reports_to_manager_offers
               JOIN managers ON managers.user_id = reports_to_manager_offers.user_id) reports_to_manager ON reports_to_manager.offer_id = offers.offer_id
  LEFT JOIN (SELECT text_field_value, entity_id FROM free_text_value WHERE text_field_name = 'offer_exception' AND entity_table_name = 'OFFERS') offer_exceptions ON offer_exceptions.entity_id = offers.offer_id
  LEFT JOIN (SELECT au.entity_id as ts_lyr_id,
                    max(ad.denied_date) as denied_date
               FROM authorisation au
               JOIN auth_detail ad ON ad.auth_id = au.auth_id
              WHERE au.entity = 'timesheet' AND ad.denied_date IS NOT NULL
              GROUP BY au.entity_id) denied_timesheet_layers ON denied_timesheet_layers.ts_lyr_id = timesheet_layer.ts_lyr_id
  LEFT JOIN (SELECT cost_set_id AS ts_lyr_id,
                    cost_snap_shot.snap_shot_id
               FROM (SELECT cost_id, adj_id, cost_type_id, cost_set_id FROM cost_lines WHERE cost_type_id = 1 AND adj_id IS NULL) cost_lines
               JOIN cost_snap_shot ON cost_snap_shot.cost_id = cost_lines.cost_id
               LEFT JOIN snap_shot ON snap_shot.snap_shot_id = cost_snap_shot.snap_shot_id
              WHERE snap_shot.snap_type_id = 6
              GROUP BY cost_set_id,
                       cost_snap_shot.snap_shot_id) snap_shot ON snap_shot.ts_lyr_id = timesheet_layer.ts_lyr_id
 WHERE
   timesheet_layer.status <> 'NOT SUBMITTED'
   AND snap_shot.snap_shot_id IN (select snap_shot_id from snap_shot where trunc(snap_date) >= to_date('07/02/2018','dd/mm/yyyy') and snap_type_id = 6)
 ORDER BY cv.first_name,
          cv.last_name,
          timesheet_layer.week_end,
          timesheet_shifts.start_date;
```
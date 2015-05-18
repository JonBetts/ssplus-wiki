Here's a script I use for batch-killing long-running sessions (e.g. `timesheet_gate_datas` queries on BNPP);

The "kill script" column generates a SQL script you can use to kill off anything that's been running for too long (3 mins in this query). There's also a "Process ID" column you can use to identify which underlying process to kill at the OS level if the db session refuses to die.

```sql
with vs as (select rownum rnum,
                          s.sid,
                          s.serial#,
                          s.status,
                          s.username,
                          s.last_call_et,
                          s.machine,
                          s.osuser,
                          s.module,
                          s.type,
                          s.terminal,
                          p.spid
                     from v$session s
                     LEFT JOIN v$process p ON s.paddr = p.addr)
         select vs.sid ,serial# serial, spid AS "Process ID",
                vs.username "Username",
                case when vs.status = 'ACTIVE'
                          then last_call_et
                     else null end "Seconds in Wait",
                'ALTER SYSTEM KILL SESSION '''||sid||','||serial#||''';' AS "-- kill script",
                vs.machine "Machine",
                vs.osuser "OS User",
                lower(vs.status) "Status",
                vs.module "Module"
           from vs
          where vs.USERNAME is not null
            and nvl(vs.osuser,'x') <> 'SYSTEM'
            and vs.type <> 'BACKGROUND'
            AND decode(vs.status, 'ACTIVE', last_call_et, 0) > 60*3 -- running for more than 3 mins
          order by module, machine, username, status, "Seconds in Wait" DESC;
```
Setup instructions to access OTRS database.

1. Add the following line to your ~/.ssh/config file.
`Host oberon`
	`LocalForward 9996 localhost:3306`
2. Follow the steps here: http://www.joellipman.com/articles/sql/access-mysql-databases-using-oracle-sql-developer.html
3. ssh oberon
4. Add a new connection under sqldeveloper
 - User - otrs
 - password - normal db
 - select mysql tab
 - hostname - localhost
 - port - 9996
 - click choose database and select otrs
5. The following queries can be run to bulk close the queues which are not actioned:

`UPDATE ticket 
SET ticket_state_id = (SELECT id FROM ticket_state WHERE name = 'bulk closed') 
WHERE queue_id IN (SELECT id FROM queue WHERE name = 'Bugs') 
  and ticket_state_id in (select id from ticket_state where name = 'new') 
  AND ticket_lock_id = (SELECT id FROM ticket_lock_type WHERE name = 'unlock');
UPDATE ticket 
SET ticket_state_id = (SELECT id FROM ticket_state WHERE name = 'bulk closed') 
WHERE queue_id IN (SELECT id FROM queue WHERE name = 'Jenkins') 
  and ticket_state_id in (select id from ticket_state where name = 'new') 
  AND ticket_lock_id = (SELECT id FROM ticket_lock_type WHERE name = 'unlock');
UPDATE ticket 
SET ticket_state_id = (SELECT id FROM ticket_state WHERE name = 'bulk closed') 
WHERE queue_id IN (SELECT id FROM queue WHERE name = 'Big Brother') 
  and ticket_state_id in (select id from ticket_state where name = 'new') 
  AND ticket_lock_id = (SELECT id FROM ticket_lock_type WHERE name = 'unlock');`
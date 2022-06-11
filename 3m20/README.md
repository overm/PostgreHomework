Я не понял, где брать базу flights. Погуглил, нашёл по ссылке какой-то дамп:

https://edu.postgrespro.ru/demo_small.zip

Там был DDL Таблицы перелётов, который я адаптировал под GreenPlum и добавил партицианирование. Получилось следующее:
``` sql

CREATE TABLE flights (
    flight_id integer NOT NULL,
    flight_no character(6) NOT NULL,
    scheduled_departure timestamp with time zone NOT NULL,
    scheduled_arrival timestamp with time zone NOT NULL,
    departure_airport character(3) NOT NULL,
    arrival_airport character(3) NOT NULL,
    status character varying(20) NOT NULL,
    aircraft_code character(3) NOT NULL,
    actual_departure timestamp with time zone,
    actual_arrival timestamp with time zone,
    CONSTRAINT flights_check CHECK ((scheduled_arrival > scheduled_departure)),
    CONSTRAINT flights_check1 CHECK (((actual_arrival IS NULL) OR ((actual_departure IS NOT NULL) AND (actual_arrival IS NOT NULL) AND (actual_arrival > actual_departure)))),
    CONSTRAINT flights_status_check CHECK (((status)::text = ANY (ARRAY[('On Time'::character varying)::text, ('Delayed'::character varying)::text, ('Departed'::character varying)::text, ('Arrived'::character varying)::text, ('Scheduled'::character varying)::text, ('Cancelled'::character varying)::text])))
)
DISTRIBUTED RANDOMLY
PARTITION BY RANGE (scheduled_departure)
SUBPARTITION BY LIST (status)
SUBPARTITION TEMPLATE
( SUBPARTITION stat_onti VALUES ('On Time'), 
  SUBPARTITION stat_dely VALUES ('Delayed'), 
  SUBPARTITION stat_depd VALUES ('Departed'), 
  SUBPARTITION stat_arrd VALUES ('Arrived'), 
  SUBPARTITION stat_schd VALUES ('Scheduled'), 
  SUBPARTITION stat_canc VALUES ('Cancelled'),
  DEFAULT SUBPARTITION stat_othr)
  (START (TIMESTAMP '2020-01-01 00:00:00+00') INCLUSIVE
   END (TIMESTAMP '2022-01-01 00:00:00+00') EXCLUSIVE
   EVERY (INTERVAL '1 month'), 
   DEFAULT PARTITION dep_othr )
;
```

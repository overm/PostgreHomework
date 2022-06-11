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

Добавилю output создания:

``` console

main.public> CREATE TABLE flights (
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
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_dep_othr" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_dep_othr_2_prt_stat_onti" for table "flights_1_prt_dep_othr"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_dep_othr_2_prt_stat_dely" for table "flights_1_prt_dep_othr"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_dep_othr_2_prt_stat_depd" for table "flights_1_prt_dep_othr"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_dep_othr_2_prt_stat_arrd" for table "flights_1_prt_dep_othr"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_dep_othr_2_prt_stat_schd" for table "flights_1_prt_dep_othr"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_dep_othr_2_prt_stat_canc" for table "flights_1_prt_dep_othr"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_dep_othr_2_prt_stat_othr" for table "flights_1_prt_dep_othr"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_2" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_2_2_prt_stat_onti" for table "flights_1_prt_2"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_2_2_prt_stat_dely" for table "flights_1_prt_2"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_2_2_prt_stat_depd" for table "flights_1_prt_2"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_2_2_prt_stat_arrd" for table "flights_1_prt_2"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_2_2_prt_stat_schd" for table "flights_1_prt_2"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_2_2_prt_stat_canc" for table "flights_1_prt_2"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_2_2_prt_stat_othr" for table "flights_1_prt_2"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_3" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_3_2_prt_stat_onti" for table "flights_1_prt_3"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_3_2_prt_stat_dely" for table "flights_1_prt_3"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_3_2_prt_stat_depd" for table "flights_1_prt_3"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_3_2_prt_stat_arrd" for table "flights_1_prt_3"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_3_2_prt_stat_schd" for table "flights_1_prt_3"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_3_2_prt_stat_canc" for table "flights_1_prt_3"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_3_2_prt_stat_othr" for table "flights_1_prt_3"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_4" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_4_2_prt_stat_onti" for table "flights_1_prt_4"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_4_2_prt_stat_dely" for table "flights_1_prt_4"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_4_2_prt_stat_depd" for table "flights_1_prt_4"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_4_2_prt_stat_arrd" for table "flights_1_prt_4"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_4_2_prt_stat_schd" for table "flights_1_prt_4"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_4_2_prt_stat_canc" for table "flights_1_prt_4"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_4_2_prt_stat_othr" for table "flights_1_prt_4"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_5" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_5_2_prt_stat_onti" for table "flights_1_prt_5"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_5_2_prt_stat_dely" for table "flights_1_prt_5"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_5_2_prt_stat_depd" for table "flights_1_prt_5"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_5_2_prt_stat_arrd" for table "flights_1_prt_5"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_5_2_prt_stat_schd" for table "flights_1_prt_5"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_5_2_prt_stat_canc" for table "flights_1_prt_5"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_5_2_prt_stat_othr" for table "flights_1_prt_5"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_6" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_6_2_prt_stat_onti" for table "flights_1_prt_6"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_6_2_prt_stat_dely" for table "flights_1_prt_6"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_6_2_prt_stat_depd" for table "flights_1_prt_6"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_6_2_prt_stat_arrd" for table "flights_1_prt_6"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_6_2_prt_stat_schd" for table "flights_1_prt_6"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_6_2_prt_stat_canc" for table "flights_1_prt_6"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_6_2_prt_stat_othr" for table "flights_1_prt_6"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_7" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_7_2_prt_stat_onti" for table "flights_1_prt_7"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_7_2_prt_stat_dely" for table "flights_1_prt_7"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_7_2_prt_stat_depd" for table "flights_1_prt_7"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_7_2_prt_stat_arrd" for table "flights_1_prt_7"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_7_2_prt_stat_schd" for table "flights_1_prt_7"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_7_2_prt_stat_canc" for table "flights_1_prt_7"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_7_2_prt_stat_othr" for table "flights_1_prt_7"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_8" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_8_2_prt_stat_onti" for table "flights_1_prt_8"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_8_2_prt_stat_dely" for table "flights_1_prt_8"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_8_2_prt_stat_depd" for table "flights_1_prt_8"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_8_2_prt_stat_arrd" for table "flights_1_prt_8"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_8_2_prt_stat_schd" for table "flights_1_prt_8"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_8_2_prt_stat_canc" for table "flights_1_prt_8"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_8_2_prt_stat_othr" for table "flights_1_prt_8"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_9" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_9_2_prt_stat_onti" for table "flights_1_prt_9"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_9_2_prt_stat_dely" for table "flights_1_prt_9"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_9_2_prt_stat_depd" for table "flights_1_prt_9"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_9_2_prt_stat_arrd" for table "flights_1_prt_9"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_9_2_prt_stat_schd" for table "flights_1_prt_9"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_9_2_prt_stat_canc" for table "flights_1_prt_9"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_9_2_prt_stat_othr" for table "flights_1_prt_9"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_10" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_10_2_prt_stat_onti" for table "flights_1_prt_10"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_10_2_prt_stat_dely" for table "flights_1_prt_10"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_10_2_prt_stat_depd" for table "flights_1_prt_10"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_10_2_prt_stat_arrd" for table "flights_1_prt_10"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_10_2_prt_stat_schd" for table "flights_1_prt_10"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_10_2_prt_stat_canc" for table "flights_1_prt_10"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_10_2_prt_stat_othr" for table "flights_1_prt_10"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_11" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_11_2_prt_stat_onti" for table "flights_1_prt_11"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_11_2_prt_stat_dely" for table "flights_1_prt_11"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_11_2_prt_stat_depd" for table "flights_1_prt_11"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_11_2_prt_stat_arrd" for table "flights_1_prt_11"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_11_2_prt_stat_schd" for table "flights_1_prt_11"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_11_2_prt_stat_canc" for table "flights_1_prt_11"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_11_2_prt_stat_othr" for table "flights_1_prt_11"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_12" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_12_2_prt_stat_onti" for table "flights_1_prt_12"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_12_2_prt_stat_dely" for table "flights_1_prt_12"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_12_2_prt_stat_depd" for table "flights_1_prt_12"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_12_2_prt_stat_arrd" for table "flights_1_prt_12"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_12_2_prt_stat_schd" for table "flights_1_prt_12"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_12_2_prt_stat_canc" for table "flights_1_prt_12"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_12_2_prt_stat_othr" for table "flights_1_prt_12"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_13" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_13_2_prt_stat_onti" for table "flights_1_prt_13"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_13_2_prt_stat_dely" for table "flights_1_prt_13"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_13_2_prt_stat_depd" for table "flights_1_prt_13"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_13_2_prt_stat_arrd" for table "flights_1_prt_13"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_13_2_prt_stat_schd" for table "flights_1_prt_13"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_13_2_prt_stat_canc" for table "flights_1_prt_13"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_13_2_prt_stat_othr" for table "flights_1_prt_13"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_14" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_14_2_prt_stat_onti" for table "flights_1_prt_14"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_14_2_prt_stat_dely" for table "flights_1_prt_14"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_14_2_prt_stat_depd" for table "flights_1_prt_14"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_14_2_prt_stat_arrd" for table "flights_1_prt_14"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_14_2_prt_stat_schd" for table "flights_1_prt_14"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_14_2_prt_stat_canc" for table "flights_1_prt_14"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_14_2_prt_stat_othr" for table "flights_1_prt_14"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_15" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_15_2_prt_stat_onti" for table "flights_1_prt_15"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_15_2_prt_stat_dely" for table "flights_1_prt_15"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_15_2_prt_stat_depd" for table "flights_1_prt_15"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_15_2_prt_stat_arrd" for table "flights_1_prt_15"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_15_2_prt_stat_schd" for table "flights_1_prt_15"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_15_2_prt_stat_canc" for table "flights_1_prt_15"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_15_2_prt_stat_othr" for table "flights_1_prt_15"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_16" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_16_2_prt_stat_onti" for table "flights_1_prt_16"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_16_2_prt_stat_dely" for table "flights_1_prt_16"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_16_2_prt_stat_depd" for table "flights_1_prt_16"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_16_2_prt_stat_arrd" for table "flights_1_prt_16"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_16_2_prt_stat_schd" for table "flights_1_prt_16"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_16_2_prt_stat_canc" for table "flights_1_prt_16"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_16_2_prt_stat_othr" for table "flights_1_prt_16"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_17" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_17_2_prt_stat_onti" for table "flights_1_prt_17"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_17_2_prt_stat_dely" for table "flights_1_prt_17"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_17_2_prt_stat_depd" for table "flights_1_prt_17"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_17_2_prt_stat_arrd" for table "flights_1_prt_17"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_17_2_prt_stat_schd" for table "flights_1_prt_17"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_17_2_prt_stat_canc" for table "flights_1_prt_17"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_17_2_prt_stat_othr" for table "flights_1_prt_17"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_18" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_18_2_prt_stat_onti" for table "flights_1_prt_18"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_18_2_prt_stat_dely" for table "flights_1_prt_18"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_18_2_prt_stat_depd" for table "flights_1_prt_18"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_18_2_prt_stat_arrd" for table "flights_1_prt_18"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_18_2_prt_stat_schd" for table "flights_1_prt_18"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_18_2_prt_stat_canc" for table "flights_1_prt_18"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_18_2_prt_stat_othr" for table "flights_1_prt_18"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_19" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_19_2_prt_stat_onti" for table "flights_1_prt_19"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_19_2_prt_stat_dely" for table "flights_1_prt_19"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_19_2_prt_stat_depd" for table "flights_1_prt_19"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_19_2_prt_stat_arrd" for table "flights_1_prt_19"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_19_2_prt_stat_schd" for table "flights_1_prt_19"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_19_2_prt_stat_canc" for table "flights_1_prt_19"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_19_2_prt_stat_othr" for table "flights_1_prt_19"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_20" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_20_2_prt_stat_onti" for table "flights_1_prt_20"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_20_2_prt_stat_dely" for table "flights_1_prt_20"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_20_2_prt_stat_depd" for table "flights_1_prt_20"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_20_2_prt_stat_arrd" for table "flights_1_prt_20"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_20_2_prt_stat_schd" for table "flights_1_prt_20"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_20_2_prt_stat_canc" for table "flights_1_prt_20"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_20_2_prt_stat_othr" for table "flights_1_prt_20"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_21" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_21_2_prt_stat_onti" for table "flights_1_prt_21"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_21_2_prt_stat_dely" for table "flights_1_prt_21"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_21_2_prt_stat_depd" for table "flights_1_prt_21"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_21_2_prt_stat_arrd" for table "flights_1_prt_21"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_21_2_prt_stat_schd" for table "flights_1_prt_21"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_21_2_prt_stat_canc" for table "flights_1_prt_21"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_21_2_prt_stat_othr" for table "flights_1_prt_21"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_22" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_22_2_prt_stat_onti" for table "flights_1_prt_22"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_22_2_prt_stat_dely" for table "flights_1_prt_22"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_22_2_prt_stat_depd" for table "flights_1_prt_22"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_22_2_prt_stat_arrd" for table "flights_1_prt_22"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_22_2_prt_stat_schd" for table "flights_1_prt_22"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_22_2_prt_stat_canc" for table "flights_1_prt_22"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_22_2_prt_stat_othr" for table "flights_1_prt_22"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_23" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_23_2_prt_stat_onti" for table "flights_1_prt_23"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_23_2_prt_stat_dely" for table "flights_1_prt_23"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_23_2_prt_stat_depd" for table "flights_1_prt_23"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_23_2_prt_stat_arrd" for table "flights_1_prt_23"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_23_2_prt_stat_schd" for table "flights_1_prt_23"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_23_2_prt_stat_canc" for table "flights_1_prt_23"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_23_2_prt_stat_othr" for table "flights_1_prt_23"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_24" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_24_2_prt_stat_onti" for table "flights_1_prt_24"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_24_2_prt_stat_dely" for table "flights_1_prt_24"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_24_2_prt_stat_depd" for table "flights_1_prt_24"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_24_2_prt_stat_arrd" for table "flights_1_prt_24"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_24_2_prt_stat_schd" for table "flights_1_prt_24"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_24_2_prt_stat_canc" for table "flights_1_prt_24"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_24_2_prt_stat_othr" for table "flights_1_prt_24"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_25" for table "flights"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_25_2_prt_stat_onti" for table "flights_1_prt_25"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_25_2_prt_stat_dely" for table "flights_1_prt_25"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_25_2_prt_stat_depd" for table "flights_1_prt_25"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_25_2_prt_stat_arrd" for table "flights_1_prt_25"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_25_2_prt_stat_schd" for table "flights_1_prt_25"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_25_2_prt_stat_canc" for table "flights_1_prt_25"
[2022-06-11 21:32:51] [00000] CREATE TABLE will create partition "flights_1_prt_25_2_prt_stat_othr" for table "flights_1_prt_25"
[2022-06-11 21:32:53] completed in 3 s 168 ms
```

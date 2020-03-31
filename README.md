# Mysql
Laboratorio de base de datos Mysql
create database utn;
use utn;

create table equipos
(
id_equipo int auto_increment not null unique,
nombre_equipo varchar(50) not null,
primary key (id_equipo)
)Engine=InnoDB;

create table jugadores
(
id_jugador int auto_increment not null,
id_equipo int not null,
nombre varchar(50) not null,
apellido varchar(50) not null,

primary key(id_jugador),
foreign key (id_equipo) references equipos (id_equipo)
)Engine=InnoDB;

create table partidos
(
id_partido int auto_increment not null,
id_equipo_local int not null,
id_equipo_visitante int not null,
fecha datetime not null,

primary key(id_partido),
foreign key(id_equipo_local) references equipos (id_equipo),
foreign key(id_equipo_visitante) references equipos (id_equipo)
)Engine= InnoDB;

create table jugadores_x_equipos_x_partidos
(
id_jugador int not null,
id_partido int not null,
puntos int not null,
rebotes int not null,
asistencias int not null,
minutos int not null,
faltas int not null,

primary key(id_jugador, id_partido), 
foreign key(id_jugador) references jugadores (id_jugador),
foreign key(id_partido) references partidos (id_partido)

)Engine= InnoDB;

1)

DELIMITER $$
CREATE PROCEDURE alta_equipo
(
 in nombre_equipo varchar(50)
)
BEGIN
insert into equipos (nombre_equipo) values (nombre_equipo);
END$$
DELIMITER 

call alta_equipo ('Velez');

2)
DELIMITER $$
CREATE PROCEDURE alta_jugador
(
 in nombre_jugador varchar(50),
 in apellido_jugador varchar(50),
 in id int
)
BEGIN
if exists (select id_equipo from equipos where id_equipo = id)
then
  insert into jugadores (id_equipo, nombre, apellido) values (id, nombre_jugador,apellido_jugador);
else
	signal sqlstate '45000' 
	SET MESSAGE_TEXT = '1';
end if;
END$$
DELIMITER 


/*************************************************************/
/* 3                                                         */
/*************************************************************/

DELIMITER $$
create procedure sp_agregar_jugador_x_nombre_equipo
(
in equipo varchar(50),
in nombre varchar(50),
in apellido varchar(50)
)
Begin
insert into jugadores(id_equipo, nombre, apellido) values ((select id_equipo from equipos where nombre_equipo = equipo), nombre, apellido);
END$$
DELIMITER 

call sp_agregar_jugador_x_nombre_equipo('Velez','laura','burla');



4)

/*************************************/
/* OPCION 1                          */
/*************************************/
create procedure sp_insertar_equipos_jugadores(pnombre_equipo varchar(50))
begin
    declare vIdEquipo int;
	insert into equipos(nombre_equipo) values(pnombre_equipo);
    set vIdEquipo = last_insert_id();
    insert into jugadores(id_equipo, nombre, apellido)
    select vIdEquipo, nombre, apellido from jugadores_temp;
    truncate jugadores_temp;
    select max(id_equipo) from equipos;
end; $$
DELIMITER;

insert into jugadores_temp (nombre, apellido) values ('Poroto','Cudero');
insert into jugadores_temp (nombre, apellido) values ('Loco','Gati');

call sp_insertar_equipos_jugadores('River');


/*************************************/
/* OPCION 2                          */
/*************************************/

create temporary table jugadores_temp(nombre varchar(50), apellido varchar(50));
DELIMITER $$
create procedure sp_insertar_equipos_jugadores(pnombre_equipo varchar(50), out vIdEquipo int)
begin
   /* declare vIdEquipo int;*/
	insert into equipos(nombre_equipo) values(pnombre_equipo);
    set vIdEquipo = last_insert_id();
    insert into jugadores(id_equipo, nombre, apellido)
    select vIdEquipo, nombre, apellido from jugadores_temp;
    truncate jugadores_temp;
   /* select max(id_equipo) from equipos;*/
end; $$
DELIMITER;

insert into jugadores_temp (nombre, apellido) values ('Poffroto','Cudero');
insert into jugadores_temp (nombre, apellido) values ('Locffo','Gati');

call sp_insertar_equipos_jugadores('SL',@id_nuevo);
select @id_nuevo;

/**************************************************/
/* OPCION HECHA POR EL PROFESOR                   */
/**************************************************/


create temporary table jugadores_temp(nombre varchar(50), apellido varchar(50));

create procedure sp_insertar_equipos_jugadores(pnombre_equipo varchar(50),)
begin,
    declare vIdEquipo int;
	insert into equipos(nombre) values(pnombre_equipo);
    set vIdEquipo = last_insert_id();
    insert into jugadores(id_equipo, nombre, apellido)
    select vIdEquipo, nombre, apellido from jugadores_temp;
    truncate jugadores_temp;
end;


5) /***********************************************************/
   /* usar alias para poder hacer dos join en una misma tabla */
  /************************************************************/
DELIMITER $$
create procedure sp_listar_partidos_x_fecha
( 
mes int,
anio int
)
begin
SELECT
    partidos.id_partido,
    plocales.nombre_equipo as Equipo_Local,
    pvisitantes.nombre_equipo as Equipo_Visitante
FROM partidos 
INNER JOIN equipos as plocales on partidos.id_equipo_local = plocales.id_equipo
INNER JOIN equipos as pvisitantes  on partidos.id_equipo_visitante = pvisitantes.id_equipo
WHERE MONTH(fecha) = mes AND YEAR(fecha) = anio;
end $$
DELIMITER ;

call sp_listar_partidos_x_fecha(04,2020);

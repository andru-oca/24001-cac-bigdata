# CAC Big data
 
### Problema: 
Nos asignan una serie de tareas en un instituto de inglés.
- Curso en 5 Niveles: 1A, 1B, 2, 3 y 4
- Nota de aprobación del examen final: 6/10 puntos
- Plantel de instructores: 20

### Pedidos desde el area :

- Migrar de un sistema de planilla de cálculo a una BBDD SQL. Nos proporcionan una copia de la
planilla con datos "sample". Normalizar la tabla y plantear el diagrama DER.
https://docs.google.com/spreadsheets/d/1QfDc073iqZa0kIs77bXOlGQqE9K1fPSlVaPoGHtD_lA/
edit?usp=drive_link
- Crear la BBDD, tablas y cargar los datos reales. SQL para carga de datos de instructores y
alumnos: https://drive.google.com/file/d/1tkG4TJOUy1eckE66nHp6-tBMvGTMijfC/view?
usp=drive_link

Utilizando la BBDD, generar los listados u obtener la información que requieren las siguientes situaciones:
- Para reducir la deserción en el nivel 1A, se solicita una lista con nombres y apellidos para contactar
vía mail a los alumnos que no se han presentado al examen final.
```
SELECT 
    a.email AS correo_electronico
,   CONCAT("<p>Hola  ", a.nombre , ", ", a.apellido, " </p>
    <p>Espero que este mensaje te encuentre bien. Te escribimos para informarte que hasta el momento no te has presentado al segundo examen del curso. Queremos recordarte que es importante completar todos los exámenes para obtener una calificación final.</p>
    <p>Te pedimos que realices el examen pendiente a la brevedad posible. Tienes aproximadamente una semana para completarlo.</p>
    <p>Si tienes alguna pregunta o necesitas ayuda, no dudes en ponerte en contacto con nosotros.</p>
    <p>Saludos cordiales, Colegio</p>
    ") AS mensaje_alumno

FROM modelado AS m
LEFT JOIN alumno  AS a
    ON a.id_alumno = m.id_alumno
LEFT JOIN nivel AS n 
    ON m.id_nivel = n.id_nivel
WHERE 1=1
    AND n.nivel = '1A' 
    AND a.nota_final IS NULL
    ;
```


- Se envían a imprimir los diplomas de los alumnos que aprobaron el último nivel. En el diploma
figuran el nombre, apellido y la nota final del alumno. Generar la lista para la impresión.
```
SELECT 
    a.email AS correo_electronico
,   CONCAT("<div class='diploma'>
        <div class='header'>
            <h1>Diploma de Excelencia</h1>
        </div>
        <div class='content'>
            <p>Este certificado se otorga a:</p>
            <h2>", a.apellido ,", ", a.nombre," </h2>
            <p>Por su destacado desempeño académico y dedicación en el curso.</p>
            <p> Calificacion final: <strong>", a.nota_final ," <strong></p>
        </div>
        <div class='signature'>
            <p>Firma del Director</p>
        </div>
    </div>

    ") AS diploma_alumno

FROM modelado AS m
LEFT JOIN alumno  AS a
    ON a.id_alumno = m.id_alumno
WHERE 1=1
    AND a.nota_final IS NOT NULL
    ;
```

- ¿Cuántos alumnos aprobaron el examen final?
```
SELECT
    COUNT(a.id_alumno) AS total_aprobados
FROM modelado AS m
LEFT JOIN alumno  AS a
    ON a.id_alumno = m.id_alumno
WHERE 1=1
    AND a.nota_final IS NOT NULL
    AND a.nota_final > 7
    ;
```


- Obtener de los alumnos inscriptos, la cantidad de aprobados, desaprobados y
ausentes por nivel.

```
SELECT
    n.nivel,
    COUNT(CASE WHEN a.nota_final >= 7 THEN 1 END) AS aprobados,
    COUNT(CASE WHEN a.nota_final < 7 AND a.nota_final >= 4 THEN 1 END) AS desaprobados,
    COUNT(CASE WHEN a.nota_final IS NULL THEN 1 END) AS ausentes
FROM alumno a
INNER JOIN modelado m ON m.id_alumno = a.id_alumno
INNER JOIN nivel n ON n.id_nivel = m.id_nivel
GROUP BY n.id_nivel, n.nivel
ORDER BY n.nivel ASC;
```

- El alumno Uta Domanek requiere revisión de su examen final y debemos contactar por email a
su instructor. Obtener su nombre, apellido y email.
```
SELECT
    i.nombre_instructor
,   i.apellido_instructor
,   i.email_instructor
,   CONCAT(
        "<h2> Revision de nota final del alumno: <strong>", a.apellido , ", ", a.nombre 
    ,   "</strong></h2>"
) AS mensaje_instructor
FROM alumno a
INNER JOIN modelado m ON m.id_alumno = a.id_alumno
INNER JOIN instructor i ON i.id_instructor = m.id_instructor
WHERE a.nombre = 'Uta' AND a.apellido = 'Domanek';

```


- El instructor designado para tomar examen final a los alumnos de último nivel que tienen apellidos
que comienzan con M,N,O y P, solicita una lista de los alumnos a examinar, ordenada
alfabéticamente.
- El instructor que obtuvo mayor cantidad de alumnos con puntaje > 9 en el examen final recibirá un
premio en la entrega de diplomas. ¿Quién es el instructor y cuántos alumnos distinguidos tuvo en
sus clases?

#### Todos los derechos son reservados por el Programa Codo a Codo perteneciente a la Subsecretaría Agencia de Aprendizaje a lo largo de la vida del Ministerio de Educación del Gobierno de la Ciudad Autónoma de Buenos Aires. Se encuentra prohibida su venta o comercialización.
# 4. Actividades del proyecto
[Ir a Readme de segunda quincena](#UD2Q2)

## Estado actual del sitio (actualizado en la quincena 3)

*Las referencias a `contacto.html` y al formulario de contacto que aparecen en las secciones históricas de este README reflejan el plan inicial de la 1.ª quincena: esa página se eliminó durante la quincena 3 (era una prueba) y volvió después sin formulario, a propósito: los formularios se estudian en el Tema 4. Detalle en `registro_ia.md`, sesiones 9 y 12.*

```text
comes_christian_tema_03/
├── README.md
├── AGENTS.md
├── registro_ia.md
├── index.html
├── proyectos.html
├── sobremi.html
├── contacto.html
├── cv-es.html
├── content/
│   ├── informes/
│   │   └── nucster/                   (informes sobre el clúster)
│   │       ├── index.html             (índice de informes técnicos)
│   │       ├── informe-cluster01.html (INF-2026-01)
│   │       ├── informe-cluster02.html (INF-2026-02)
│   │       └── informe-validacion.html (INF-2026-03)
│   └── posts/
│       ├── nucster/
│       │   ├── cluster01elgermen.html
│       │   ├── cluster02primerospasos.html
│       │   └── inventario.html        (acceso directo histórico al inventario de hardware)
│       └── portfolio/                 (serie sobre la elaboración del sitio)
│           ├── portfolio01planificacion.html
│           ├── portfolio02construccion.html
│           └── portfolio03validacion.html
├── img/
│   ├── ChristianComes.jpg
│   ├── estanterias-proyectos.svg
│   ├── forest-gump-wave.gif
│   ├── nucster01.jpg
│   └── cluster01/
│       ├── switch.png
│       ├── nucster10/   (sumary, fastfetch, vlan_aware, cluster, disks)
│       ├── nucster20/   (summary, fastfetch, vlan_aware, cluster, disks)
│       └── nucster30/   (summary, fastfetch, vlan_aware, cluster, disks)
├── vid/
│   └── nucster/
│       └── 02/
│           └── demostracion_cluster.webm
├── docs/
│   ├── memoria-proyecto.md            (cuestionario Parte A de la Quincena 3)
│   └── auditorias/
│       └── accesibilidad-tema3.md
├── src/
│   └── CV/                         (currículums, no evaluable)
│       ├── Christian_Comes_L2_Incidents_CV_ES.pdf   (CV en español)
│       └── Christian_Comes_L2_Incidents_CV_EN.pdf   (CV en inglés)
├── evidencias/
│   ├── ud2/   (validador, uso de IA, capturas)
│   └── ud3/   (validador W3C, teclado, sin imágenes)
└── ContextoIA/
    └── memoriaagente.md
```

**Navegación:** el menú principal tiene 6 elementos fijos — Inicio · Sobre mi · Contacto · Proyectos · Inventario · CV. Las entradas del blog no van en el menú: se enlazan desde el apartado «Entradas Recientes» al pie de cada entrada y desde `proyectos.html`.

## Actividad 1. Tema elegido y justificación

El tema es un blog de tipo portfolio hosteado en abierto en **GitHub**, en el que documentaré paso a paso desde cero la creación de un clúster de 3 mini PC con **Proxmox** para aprender sobre **Kubernetes** y aplicar los conocimientos adquiridos durante el ciclo formativo.

La justificación es sencilla: cuento con años de experiencia en soporte de IT y una afición personal por la gestión de *homeservers*. Certificar ese conocimiento con este ciclo es importante para mi progresión laboral. Los reclutadores de IT valoran mucho la demostración práctica de capacidades; montar servidores y documentar todo el proceso demuestra competencia en administración de sistemas y capacidad de documentación técnica correcta.

---

## Actividad 2. Mapa inicial de información del proyecto

La estructura inicial consta de las siguientes secciones:

1.  **Home / About me:** Información sobre el proyecto y mi perfil personal, incluyendo acceso directo a la primera y a la última entrada del blog (para facilitar tanto el seguimiento cronológico como el acceso a las novedades).
2.  **Blog:** Sección principal con capacidad de ordenar el contenido de forma ascendente y descendente.
3.  **Formulario de contacto:** Canal directo para reclutadores o interesados.
4.  **Sección de proyectos:** Dividida inicialmente en dos grandes bloques: **Homelab** y **Sitio web**. Ambos se subdividen en temas específicos documentados bajo dos enfoques:
    *   *Blog entretenido:* Formato distendido para divulgación.
    *   *Informes de IT:* Documentación quirúrgica, aséptica y técnica.

> **Nota de actualización (quincena 3):** el punto 3 de este mapa (formulario de contacto) se eliminó del sitio; la navegación actual es Inicio · Sobre mi · Proyectos. Detalle en `registro_ia.md`, sesión 9.

**Elementos clave de la infraestructura:**
*   Inventario de infraestructura del clúster.
*   Diagrama de red e infraestructura (con histórico consultable).
*   Diagrama de servicios hospedados (con histórico consultable).
*   Captura de *dashboard* de estado del clúster y servicios (Pendiente de investigación).

---

## Actividad 3. Inventario de formatos que podrían intervenir

| Formato o tecnología | Uso previsto |
| :--- | :--- |
| **HTML** | Estructura de las páginas, entradas y páginas fijas del sitio. |
| **CSS** | Diseño visual y adaptación a dispositivos móviles (Responsive). |
| **JavaScript** | Ordenación de entradas, búsqueda, filtros o navegación dinámica. |
| **Markdown** | Redacción de la documentación técnica y de las entradas del blog. |
| **PNG, JPG o WebP** | Capturas del proceso, gráficos e imágenes ilustrativas. |
| **SVG** | Diagramas de red y arquitectura escalables. |
| **JSON** | Datos de las entradas, proyectos o métricas de ejemplo. |
| **XML** | Representación estructurada de recursos y prácticas de validación. |
| **RSS o Atom** | Sindicación de las nuevas entradas del blog. |
| **YAML** | Archivos de configuración de Kubernetes y despliegues. |

---

## Actividad 4. Glosario inicial y autoevaluación

### Respuestas de autoevaluación

*   **¿Qué parte del proyecto está menos clara?**
    Sinceramente, su forma final. Tenía una visión muy simplista, pero cuanto más indago en la elaboración del sitio, más posibilidades técnicas y de contenido descubro.
*   **¿Qué decisión técnica desea validar con el profesor?**
    ¿Cómo procedo a compaginar Markdown como herramienta de formato de texto y HTML para organizar la información de las entradas de forma eficiente?
*   **¿Qué patrón del ejemplo KDP Manager traslada al proyecto?**
    *   **Obra de KDP = Entrada técnica/servicio:** Cada proyecto principal (ej. Clúster Proxmox, Router pfSense) equivale a una "obra".
    *   **Ficha de una obra = Ficha de recurso:** Especificación de hardware, SO, IPs y estado de cada MV o nodo.
    *   **Capítulo / Versión = Entrada de bitácora:** Pasos acumulativos del proceso documentado.
    *   **Formatos (HTML, XML, JSON, MD) → Vistas:** HTML para presentación, XML para validación, JSON para contratos de API y Markdown para documentación.
*   **¿Qué cambiaría si crecieran los usuarios, las visitas o los contenidos?**
    En la estructura web nada; sería cuestión de escalar el hosting o valorar el paso a un plan de pago si el volumen de peticiones supera los límites gratuitos.

### Estructura de proyecto propuesta

```text
homelab-portfolio/
├── README.md
├── content/
│   ├── posts/
│   │   └── bitacora-01-proxmox.md
│   └── proyectos/
│       └── cluster-nuc6.md
├── docs/
│   ├── diagramas/
│   │   └── topologia-red.svg
│   └── inventario-infraestructura/
│       └── nodos.csv
├── datos/
│   ├── nodos.json
│   └── infra.xml
├── evidencias/
│   ├── validacion_w3c.png
│   └── captura_segura.png
├── img/
│   └── diagrama-cluster.png
├── index.html
├── sobremi.html
├── contacto.html
└── proyectos.html
```

> **Nota de actualización:** esta es la estructura propuesta inicialmente en la 1.ª quincena; `contacto.html` se eliminó en la quincena 3 y la estructura vigente es la de [Estado actual del sitio](#estado-actual-del-sitio).

---

# UD2Q2

## Actualización del README.md a para la entrega de la segunda quincena

### Estructura actual de carpetas, por el momento.

_Esta estructura está simplificada según requisito de entrega de tema 2, pero empieza a mostrar el formato que más adelante se explica._

```text
homelab-portfolio/
├── README.md
├── registro_ia.md
├── content/
│   └── posts/
│       └── nucster
│           └── cluster01elgermen.html
│ 
├── evidencias/
│   ├── Index_validador_primerintento.png
│   ├── proyectos_validador_primerintento.png
│   ├── cluster01_validador_primerintento.png
│   ├── ChristianComes_cluster01elgermen_RevisionIA03.png
│   ├── ChristianComes_cluster01elgermen_RevisionIA02.png
│   ├── ChristianComes_cluster01elgermen_RevisionIA.png
│   ├── ChristianComes_Proyectos_RevisionIA.png
│   └── ChristianComes_Index_RevisionIA.png
├── img/
|   ├── forest-gump-wave.gif
|   ├── estanterias-proyectos.svg
│   └── nucster01.png
├── index.html
└── proyectos.html
```

### Preguntas de Autoevaluación

1. **Puedo explicar qué conceptos del manual he aplicado.**

    He aplicado la semántica para organizar las estructuras de las diferentes páginas web con la intención de ayudar a su catalogación, teniendo en cuenta no usar etiquetas en función de su impacto en la representación visual por encima de su correcto uso semántico funcional.

2. **Puedo señalar qué parte del proyecto es nueva respecto a la quincena anterior.**

    Con la ayuda del profesor y sus correcciones he podido definir mejor la estructura de carpetas. Todo esto es muy nuevo para mí y la primera quincena iba totalmente a ciegas. Es cierto que debido a mi ignorancia en este campo aún me cuesta decidir que la estructura actual sea la definitiva puesto que voy descubriendo nuevas secciones potenciales que agregar al sitio y tal vez suponga alterar la estructura de carpetas.

3. **Puedo justificar al menos tres decisiones técnicas.**

    1. No usar la etiqueta `<ol>` por motivos estéticos en el front. La semántica HTML establece que los listados que utilizan `<ol>`solo se utilizan para listados secuenciales, como en instrucciones de montaje o recetas de cocina.

    2. No utilizar `<br> ni <p>` para maquetar el front end. Se debe respetar la semántica en todo momento, y eso incluye saber por ejemplo que `<p>`se utiliza para representar párrafos distintos cuyo contenido difiere lo suficiente como para considerarse tema a parte del que precedía, tal y como se determina en la ortografía española.

    3. Mantener las páginas de plantilla fija en la raíz del sito y las de contenido de prosa en sus propias carpetas. Páginas como Index.html, Proyectos.html; en adelante: sobremi.html, contacto.html [contacto.html fue eliminado en la quincena 3, ver `registro_ia.md`]; son páginas genéricas que podrán ser re-redactas en cuanto a contenido pero son de acceso más rápido, las entradas de blog, y sus versiones de informes (cuando empiece con ellas en este curso) acabarán siendo cuantiosas y tenerlas en la raíz supone un caos descontrolado para su mantenimiento en el back-end por lo que se ubican en sus propias carpetas. Por eso las carpetas de este contenido van a estar subdivididas por proyecto (homelab y he decidido documentar también este sitio web).

4. **He probado mi resultado y he corregido los errores encontrados.**

    Correcto, llevo días con esto, y he revisado y corregido los errores semánticos y funcionales que han ido ocurriendo, especial mención a la navegación relativa al inicio tras poner el primer post en una carpeta en lugar de la raíz.
5. **Puedo reproducir la entrega desde cero.**

    Puede que no punto por punto y carpeta por carpeta, pero al haber hecho este trabajo yo mismo, y no una IA, el sería capaz de replicarlo de memoria al 90%.

6. **Puedo adaptar el ejemplo de KDP Manager a mi propio dominio sin copiarlo.**

    Hasta cierto punto es lo que he hecho, utilizar los ejemplos de index y obra, ha sido de gran utilidad para comprender el por qué de unas etiquetas y no otras, así como su relación.
    La correlación sería:

    1. Index.html (original) es Index.html en mi versión.

    2. catalogo.html (original) se convierte en proyectos.html una página con formato de biblioteca donde encontrar todas las entradas de los proyectos documentados.

    3. obra.html es cluster01elgermen.html una entrada de blog o un informe.
  

### Preguntas para la tutoría
1. **¿Qué parte de mi proyecto está menos clara o peor estructurada?**

    Cómo es mejor ir documentando la evolución del sitio web.

2. **¿Qué decisión técnica necesito validar con el profesor?**

    Si mi motivo y forma de decidir .md para entradas e informes es correcta o no.

3. **¿Qué elemento del ejemplo KDP Manager he trasladado a mi dominio y por qué?**

    El respeto por la semántica, y la estructura de las páginas, hasta hoy para mi las páginas de dividian en head, body, footer y el resto de etiquetas, y solo se utilizaban para dar formato visual a los sitios web. Nadie me había explicado que las etiquetas tienen como función principal catalogar la información por su tipo.

4. **¿Qué cambiaría si el número de usuarios o datos creciera considerablemente?**

    El ignorante que hay en mí dice que eso depende de dónde hospede el sitio, si en un servidor propio, o en un host ajeno, pero básicamente tendría que aumentar los recursos de hardware para soportar el tráfico.
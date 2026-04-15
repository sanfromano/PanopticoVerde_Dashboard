# AI Log - Equipo Panóptico Verde

## Herramientas utilizadas
- Claude.ai en navegador web

## Filosofía de uso
Se utilizó para acelerar la parte técnica, se solicita código al que se le pueda adapatar o reusar a lo largo del proyecto para diferentes fuentes de datos y datos.
Se utiliza para evaluar viabilidad o estructura de ideas.
El código se revisa y siempre es copiado y pegado a mano o escrito a mano usando como referencia el generado, sin uso de agentes de IA, solo chatbots.
## 7 abril 2026
Se comparte documentación de https://agenda2030.mx/ODSopc.html?lang=es#/massive
a partir de la cual se construye un explorador de la API, generando el notebook presente como ods_explorer el cual es sometido a reescritura tanto humana como asistida por IA.

## 13 abr 2026
Tarea: Construir código base para generar fábrica de peticiones a utilizar
Prompt:
En un inicio trabajaremos en Notebooks de Python, ya luego integraremos a quarto.
Necesito poder obtener data de ciertos indicadores, así que haremos esto:
-Crear una factory para construir nuestros requests de datos con la url base + el inidicador que queremos
-Usar si es posible las mismas funciones para obtener datos de INEGI.
-Guardar los datos 
Resultado: Generó métodos genéricos que en conjunto con métodos específicos para cada API peticiona y guarda los datos en un cache 
Decisión: Se trabajó con esa base para el peticionado de datos al tener una base sólida sobre la cual se puede expandir las peticiones a diferentes API's

## 13 abr 2026
Tarea: Convertir los datos peticionados en algo utilizable para generar gráficas
Prompt: 
Vamos a pintar, veo que tenemos definidas las funciones para los overlays que nos son muy importantes, lo que quisiera es que en la ejecución principal pasara lo siguiente:
-Elegimos si usamos como fuente un .csv o lo que saquemos live de las api's (en cuyo caso, será hacer la llamada ahí en caliente, no?)
-Elegimos los indicadores que ya sea que llamemos o tomemos del caché en adelante
-Pongamos las funciones para dibujar las gráficas que queremos.
Resultado: Generó código que utiliza la fábrica de peticiones que hicimos previamente al solicitarlos en una lista o al descomentar una línea usa el caché. También generó código para generar algunas gráficas.
Decisión: Se preserva el código para solicitar los datos, se separa el código para las gráficas.
## 13 abrl 2026
Tarea: Explorar API's de la ONU y su cobertura
Prompt:
Ahora haremos algo divertido, quiero mapear que ODS están y cuales no dentro de una fuente específica, partamos de la de la onu, aún sin código, cómo podríamos explorarlo e interpretarlo? la falta de datos nos parece interesante de mostrar en nuestra narrativa 
Respuesta: Propone hacer un barrido de la API de la ONU y mapear el tamaño de las respuestas
Decisión: Aceptar esa propuesta
Resultado: Propone varios experimentos y tras un ir y venir con el usuario, se llega a la solución de hacer un barrido de 251 indicadores de manera incremental separado en dos celdas, de manera que si se interrumpe la ejecución se puede continuar desde donde se dejó.
Decisión: Se queda el código que hace el barrido

## 13 abril 2026
Tarea: Construir gráfica que segemente NEET en hombres y mujeres como barras apiladas y superponga el índice de Gini
Resultado: Código que genera lo solicitado. 
Decisión: se mantiene y se usa como base para futuras gráficas


## 14 abril 2026
Tarea: Graficar la cobertura de la API de la ONU
Prompt: Super, mientras ejecuta quisiera poder visualizarlo, pienso como un mapa de calor de las metas que están mejor o peor documentadas
Resultado: código con un mapa de calor de la cobertura.
Decisión: Se conserva al ser de relevancia a la narrativa y justo lo que se solicitó

## 14 abril 2026
Tarea: Se utiliza el Claude para consolidar y corregir el código para su entrega. Se mantiene conversación constante para hacer arreglos funcionales y estéticos
## 14 abril 2026
Tarea: Generar un log de ia en xml para contextualizar otros agentes/chats.
Prompt:Porfavor, vamos a hacer un XML que se pueda usar para dar contexto de lo que hemos trabajado a otra instancia de Claude u  otra IA, esto para complementar con el compatir el notebook en si.
Decisión: Se elimina algunos datos irrelevantes o de ideas pasadas y se deja lo siguiente documentado
Resultado:
<?xml version="1.0" encoding="UTF-8"?>
<project_context>

  <metadata>
    <title>SDG Indicators Explorer — Proyecto de investigación ODS México</title>
    <stage>Experimental / notebook Python</stage>
    <next_stage>Dashboard interactivo en Quarto</next_stage>
    <repo_structure>
      <path>notebooks/sdg_indicators.ipynb</path>
      <path>notebooks/data_cache/</path>
    </repo_structure>
  </metadata>

  <research_purpose>
    Explorar correlaciones entre indicadores ODS para identificar narrativas de mejora
    integral basadas en casos reales de transformación comunitaria autogestionada.
    La hipótesis central es que intervenciones locales (no políticas centralizadas)
    generan mejoras simultáneas en múltiples ODS. El dashboard final permitirá
    simular escenarios: "si X mejora n%, se proyecta una mejora de m% en Y".
    
  </research_purpose>

  <architecture>

    <cell name="imports_and_config">
      <description>Imports globales, configuración de paths y constantes</description>
      <key_variables>
        <var name="CACHE_DIR" type="Path" value="data_cache/" />
        <var name="YEAR_START" type="int" value="2000" />
        <var name="YEAR_END" type="int" value="2023" />
        <var name="ONU_SDG_BASE" type="str" value="https://unstats.un.org/sdgs/UNSDGAPIV5/v1" />
        <var name="ONU_AREA_CODE" type="str" value="484" note="ISO-3166 numérico de México" />
      </key_variables>
    </cell>

    <cell name="indicator_dataclass">
      <description>Dataclass Indicator — representa un indicador de cualquier fuente</description>
      <fields>
        <field name="code" type="str" note="código en la API origen" />
        <field name="label" type="str" note="nombre corto para gráficas" />
        <field name="source" type="str" note="worldbank | inegi | onu_sdg" />
        <field name="unit" type="str" />
        <field name="country" type="str" default="MEX" />
        <field name="extra_params" type="dict" note="ej. token INEGI" />
      </fields>
    </cell>

    <cell name="catalog">
      <description>
        Dict CATALOG — punto de entrada para agregar indicadores.
        Solo se edita esta celda para agregar fuentes nuevas.
        Cada key es un alias corto, el valor es un Indicator.
      </description>
      <sources_mapped>
        <source name="worldbank">
          <indicators>
            <item key="neet"                code="SL.UEM.NEET.ZS"      ods="8.6.1" />
            <item key="neet_f"              code="SL.UEM.NEET.FE.ZS"   ods="8.6.1" />
            <item key="neet_m"              code="SL.UEM.NEET.MA.ZS"   ods="8.6.1" />
            <item key="gini"                code="SI.POV.GINI"          ods="10.1.1" />
            <item key="suicidio"            code="SH.STA.SUIC.P5"       ods="3.4.2" />
            <item key="mujeres_directivas"  code="SL.EMP.SMGT.FE.ZS"   ods="5.5.2" />
            <item key="acceso_carretera"    code="IS.ROD.PAVE.ZS"       ods="9.1.1" note="proxy" />
            <item key="ghg_total"           code="EN.ATM.GHGT.KT.CE"   ods="13.2.2" note="proxy" />
            <item key="ingreso_40_inferior" code="SI.SPR.PC40.ZG"       ods="10.2.1" note="proxy" />
            <item key="pobreza_extrema"     code="SI.POV.DDAY"          ods="1.1.1" />
          </indicators>
        </source>
        <source name="onu_sdg">
          <indicators>
            <item key="onu_neet"               code="8.6.1" />
            <item key="onu_suicidio"           code="3.4.2" />
            <item key="onu_mujeres_directivas" code="5.5.2" />
            <item key="onu_ghg"                code="13.2.2" />
            <item key="onu_ingreso_mediana"    code="10.2.1" />
            <item key="onu_acceso_carretera"   code="9.1.1" />
          </indicators>
        </source>
        <source name="inegi">
          <note>Requiere token gratuito de inegi.org.mx/app/api/ficha.html</note>
          <note>Pasar como extra_params={"token": "TU_TOKEN"} en el Indicator</note>
        </source>
      </sources_mapped>
    </cell>

    <cell name="data_factory">
      <description>
        Clase DataFactory — dispatcher central. Toda la lógica de fetch,
        caché en disco (JSON), exportación e importación CSV vive aquí.
        Para agregar una fuente nueva: añadir método fetch_nueva_fuente()
        dentro de la clase y un elif en el dispatcher fetch().
        IMPORTANTE: todos los métodos deben estar en una sola celda —
        no se pueden definir métodos de una clase en celdas separadas en Jupyter.
      </description>
      <methods>
        <method name="fetch_worldbank" note="usa wbgapi, maneja paginación automática" />
        <method name="fetch_inegi" note="BIE API, requiere token" />
        <method name="fetch_onu_sdg">
          <note>
            dimensions en la respuesta es dict plano {Age: 15-24},
            NO lista de dicts. Filtro de Sex/Location con fallback
            si el indicador no tiene esas dimensiones.
          </note>
        </method>
        <method name="fetch" note="dispatcher — despacha según indicator.source" />
        <method name="fetch_many" note="itera CATALOG, retorna dict[str, DataFrame]" />
        <method name="to_csv" note="exporta dict de DataFrames a data_cache/" />
        <method name="from_csv" note="carga CSVs desde data_cache/ sin hacer requests" />
      </methods>
      <output_contract>
        Todo método fetch_* debe retornar DataFrame con columnas:
        year (int), value (float), label (str), unit (str), source (str)
      </output_contract>
    </cell>

    <cell name="visualization">
      <description>Funciones de graficación con Plotly. Reciben dict[str, DataFrame].</description>
      <functions>
        <function name="plot_overlay">
          <note>Líneas superpuestas con eje Y por indicador (distintas unidades)</note>
          <note>Toggle de visibilidad gratis en la leyenda de Plotly</note>
        </function>
        <function name="plot_grid">
          <note>Cada indicador en su propio subplot — útil para exploración</note>
        </function>
        <function name="build_heatmap">
          <note>Heatmap de disponibilidad de indicadores por ODS</note>
          <note>Reutilizable para cualquier fuente — recibe resultados de barrido</note>
          <note>
            Usa indexing vectorizado numpy, NO iterrows.
            Filtra goal_num entre 1 y 17 antes de construir la matriz.
          </note>
          <parameters>
            <param name="resultados" type="list[dict]" note="output de barrido" />
            <param name="catalogo_meta" type="dict" note="{code: {goal, description}}" />
            <param name="title" type="str" />
          </parameters>
        </function>
      </functions>
    </cell>

    <cell name="barrido_disponibilidad">
      <description>
        Exploración de cobertura de indicadores ODS por fuente para México.
        Guarda progreso incremental cada 10 indicadores.
      </description>
      <functions>
        <function name="get_all_indicators" note="catálogo ONU — 251 indicadores" />
        <function name="check_indicator" note="checa disponibilidad en ONU SDG API" />
        <function name="check_indicator_wb" note="checa disponibilidad en World Bank" />
        <function name="barrido_to_csv" note="exporta resultados de barrido a CSV" />
        <function name="barrido_from_csv" note="carga barrido desde CSV" />
      </functions>
      <cache_files>
        <file name="barrido_onu.json" note="progreso incremental del barrido ONU" />
        <file name="barrido_wb.json"  note="progreso incremental del barrido WB" />
        <file name="barrido_onu.csv"  note="resultado final barrido ONU" />
        <file name="barrido_wb.csv"   note="resultado final barrido WB" />
      </cache_files>
      <status_values>
        <value name="completo" score="2"  note="10 o más años con datos" />
        <value name="parcial"  score="1"  note="entre 1 y 9 años" />
        <value name="ausente"  score="0"  note="0 observaciones" />
        <value name="error"    score="-1" note="fallo en el request" />
      </status_values>
    </cell>

    <cell name="ejecucion_principal">
      <description>
        Celda de orquestación — único lugar que se edita para cambiar qué se corre.
        FUENTE = "live" hace requests a las APIs y guarda CSV.
        FUENTE = "csv" carga desde data_cache/ sin requests.
      </description>
    </cell>

  </architecture>

  <data_sources>
    <source name="World Bank API">
      <base_url>https://api.worldbank.org/v2</base_url>
      <library>wbgapi</library>
      <country_code>MEX</country_code>
      <note>Códigos internos WB, no siguen formato ODS oficial</note>
      <note>Mapeo manual ODS → WB en dict ODS_WB_METAS (~55 metas)</note>
    </source>
    <source name="ONU SDG API">
      <base_url>https://unstats.un.org/sdgs/UNSDGAPIV5/v1</base_url>
      <country_code type="ISO-3166-numeric">484</country_code>
      <catalog_size>251 indicadores</catalog_size>
      <note>Códigos ODS oficiales (8.6.1, 5.5.2, etc.)</note>
      <note>dimensions en respuesta es dict plano, no lista de dicts</note>
      <note>Cobertura irregular para México — verificar antes de usar</note>
    </source>
    <source name="INEGI BIE">
      <base_url>https://www.inegi.org.mx/app/indicadores/ws/series/BISE</base_url>
      <note>Requiere token gratuito — pendiente de integrar</note>
      <note>Series identificadas: 493911 PIB, 444612 desocupación, 216064 INPC</note>
    </source>
  </data_sources>

  <known_issues>
    <issue>
      fetch_onu_sdg — dimensions es dict plano {Age: 15-24}, no lista.
      Versión anterior asumía lista de dicts y lanzaba TypeError.
    </issue>
    <issue>
      DataFactory — todos los métodos deben estar en una sola celda Jupyter.
      Definir métodos en celdas separadas los convierte en funciones sueltas
      y la clase no los reconoce como propios.
    </issue>
    <issue>
      build_heatmap — usar indexing vectorizado numpy en lugar de iterrows.
      iterrows es muy lento para matrices grandes.
      Filtrar goal_num.between(1, 17) antes de construir la matriz
      para evitar IndexError por códigos WB con formato distinto.
    </issue>
    <issue>
      World Bank topic=17 devuelve 29,511 indicadores con códigos internos.
      No hay campo que mapee automáticamente a códigos ODS oficiales.
      Solución: mapeo manual en ODS_WB_METAS (~55 metas principales).
    </issue>
  </known_issues>

  <pending>
    <item>Completar ODS_WB_METAS con las ~55 metas y correr barrido WB completo</item>
    <item>Integrar token INEGI y correr barrido de esa fuente</item>
    <item>Heatmap comparativo ONU vs WB con cobertura completa</item>
    <item>Visualización barras apiladas cobertura WB por ODS</item>
    <item>Análisis de correlaciones entre indicadores (Pearson, clustering)</item>
    <item>Contextualización con eventos históricos (series temporales + marcadores)</item>
    <item>Simulador de escenarios: intervención en ODS X → impacto en ODS Y</item>
    <item>Migración de notebook a dashboard Quarto con Plotly</item>
    <item>Pipeline RSS → LLM para feed de noticias relevantes (pausado)</item>
  </pending>

</project_context>
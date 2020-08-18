
# Table of Contents

1.  [Overview](#org3cdde55)
2.  [Report Context](#org7e9d067)
    1.  [Overview](#org5a7e5f0)
    2.  [data](#org1930a84)
    3.  [workflows](#org00433c0)
3.  [Fishing Area Context](#orgda4830a)
    1.  [Overview](#org6d34a9f)
        1.  [Goals](#org9d23a7c)
    2.  [data](#org975a529)
    3.  [workflows](#org17d6214)
        1.  [substeps](#org4114593)
4.  [Vessel Identification Context](#org9828b43)
    1.  [Overview](#orgd5ee7ef)
        1.  [Goals](#org798d796)
    2.  [Design trade-offs](#org5525839)
    3.  [data](#org84828d4)
5.  [Informer Data Context](#org988d96e)
    1.  [Overview](#orgd02870b)
        1.  [Goals](#org66ef773)
    2.  [data](#org1112e3d)
6.  [Help Context](#orge32162e)
    1.  [Overview](#org949e85e)
        1.  [Goals](#org2e7ebbf)
7.  [Scratch](#orgc876d40)


<a id="org3cdde55"></a>

# Overview

![img](components-app.png)


<a id="org7e9d067"></a>

# Report Context


<a id="org5a7e5f0"></a>

## Overview

La flota pesquera artesanal denuncia que se producen encuentros en alta mar con
embarcaciones sospechosas de ejercer pesca ilegal. Principalmente flota de
bandera asiática. Carecen de herramientas específicas que les permitan registrar
estos encuentros y no existe un consenso sobre cuál es la información más
valiosa a capturar.

Estos encuentros se producen generalmente en zonas fuera de cobertura móvil y la
estancia en esas zonas puede durar hasta 8 días.


<a id="org1930a84"></a>

## data

    data Report =
      FishingVessel
      AND NoEmptyList of SuspiciousActivity
      AND InformerData


<a id="org00433c0"></a>

## workflows

    workflow "ReportSuspiciousActivity" =
      input = EmptyReport
      output (on success) = SuspiciousActivityReported
      output (on error) = InvalidReport
    
    //step 1
    do CheckFields


<a id="orgda4830a"></a>

# Fishing Area Context


<a id="org6d34a9f"></a>

## Overview

El proyecto va dirigido al ámbito de Calamasur. Los límites de las zonas
económicas exclusivas (EEZ) donde se producen el mayor número de encuentros
están definidos y aceptados internacionalmente.

Otras zonas de pesca tendrán problemas que serán total o sutilmente diferentes a
los de Calamasur, y sobre los que SFP no tiene actualmente acuerdos o competencias.
No es descartable que en futuras versiones se de soporte a estos otros problemas.

Todos los dispositivos son capaces de obtener su geolocalización.


<a id="org9d23a7c"></a>

### Goals

-   Mejorar la experiencia de usuario con una interfaz y reglas de negocio
    adaptadas a la localización.
-   Comprobar que las capacidades de localización del dispositivo están encendidas.
-   Garantizar que al comienzo del proceso tenemos una posición minimamente
    precisa.
-   Minimizar el número de informes no válidos que moderar (Desde tierra, zonas
    sin competencias&#x2026;)


<a id="org975a529"></a>

## data

    data DeviceLocationData =
      Latitude
      AND Longitude
      AND Accurancy
    
    data DesiredAccurancy =
      Meters
    
    data GetFishingAreaInput =
      DeviceLocationData
      AND DesiredAccurancy
    
    data AccurateLocation =
      AccurateLatitude
      AND AccurateLongitude
    
    data SupportedAreas =
      list of SupportedArea
    
    data SupportedArea =
      FishingAreaName
      AND PolygonArea // As in marineregions.org
    
    data SupportedFishingArea =
      CalamasurArea
      OR GenericArea
      OR ...


<a id="org17d6214"></a>

## workflows

    workflow "GetFishingArea" =
      input: GetFishingAreaInput
      output (on success): SupportedFishingAreaDetected
      output (on error): UnsupportedFishingArea OR InaccurateLocation
    
    //step1
    do CheckLocationAccurancy
      If inaccurate
      return InaccurateLocation
      stop
    
    //step2
    do CheckPosition
      If unsupported
      return UnsupportedFishingArea
      stop
    
    //step3
    return SupportedFishingAreaDetected


<a id="org4114593"></a>

### substeps

    substep "CheckLocationAccuracy" =
      input: GetFishingAreaInput
      output (on success): AccurateLocation
      output (on error): InaccurateLocation
    
    substep "CheckPosition" =
      input: AccurateLocation
      dependency: SupportedAreas
      output (on sucess): SupportedFishingArea
      output (on error): UnsupportedFishingArea


<a id="org9828b43"></a>

# Vessel Identification Context


<a id="orgd5ee7ef"></a>

## Overview

En Calamasur las embarcaciones sospechosas de ejercer pesca ilegal son
principalmente de bandera asiática.

Puede ser particularmente difícil identificar los barcos asiáticos por sus
nombres, ya que existe una gran variación en la forma en que se escriben los
caracteres chinos en letras romanas, por lo que capturar cualquier número
visible en el casco es particularmente importante.

Hay diversidad de opiniones respecto a la cercanía de los encuentros y la
capacidad de poder capturar información relevante con el dispositivo móvil. Sin
embargo parece unánime que los buques sospechosos suelen estar muy sucios,
nombre ilegible o intentos de ocultar sus marcas.

Según FAO los intentos de ocultar sus marcas son en si mismo un motivo de denuncia,
independientemente de la actividad pesquera que estén ejerciendo. Lo mismo para
marcas duplicadas, o navegar sin bandera.

Las embarcaciones sospechosas ejercen una actividad industrial, por lo que estan
obligados a emitir por el sistema de radio AIS. Sin embargo las embarcaciones
artesanas de Calamasur en su mayoría no disponen de receptores AIS. Si disponen
de AIS las flotas industriales de la zona.


<a id="org798d796"></a>

### Goals

-   Obtener la información mas relevante posible
-   Obtener avisos aun sin información relevante
-   Educar sobre los diferentes hechos denunciables, mas alla de la actividad
    pesquera.
-   Tratar de obtener pruebas gráficas, aun de calidad baja, donde se capturen
    detalles de la construcción de los buques.
-   Llegar a conocer si el sistema AIS está apagado o enviando datos falsos (spoofing).


<a id="org5525839"></a>

## Design trade-offs

TODO: número de fotos permitidas
TODO: número de vídeos


<a id="org84828d4"></a>

## data

    // Markings
    data IMONumber =
      string 9 numbers
    
    data CallSign =
      string first 3 chars as ITU codes
    
    data Other =
      string
    
    data Marking <a> =
      <a>
      OR <a> MarkingNotVisibleOrHidden
    
    data Markings =
      Marking<CallSign>
      AND Marking<IMONumber>
      AND list of Marking<Others>
    
    
    // AIS Transmission
    data MMSI =
      string 7 numbers
    
    data AISDeclaredActivity =
      Fishing
      OR ToPort
      OR ...
    
    data AISTransmission =
      MMSI
      AND AISCallSign
      AND AISName
      AND AISDeclaredActivity
    
    
    // Vessel
    data FishingVessel =
      Type
      AND Markings
      AND AISTransmission
      AND list of Photos
    
    // Activity
    data SuspiciousActivity =
      EEZFishingActivity
      OR HiddingMarkings
      OR DuplicatedMarkings
      OR SignsOfTampering
      OR Stateless
      OR NotAISSignal
      OR AISSpoofedSignal
      OR Other


<a id="org988d96e"></a>

# Informer Data Context


<a id="orgd02870b"></a>

## Overview

El usuario o informador será generalmente un patrón de embarcación de pesca
artesanal. Suele ser reticente a desvelar su identidad o a hacer pública la
posición geográfica exacta del encuentro.

La flota artesanal carece de transmisores AIS, pero está en plena implantación
del sistema VMS. La flota industrial si transmite AIS públicamente.

Los dispositivos móviles Android que utilizan permiten obtener, no sólo la
posición, si no también volcados raw de la información de los sistemas de
navegación.


<a id="org66ef773"></a>

### Goals

-   Extraer el máximo contexto posible del momento de la captura
-   Tener la información necesaria para poder dar veracidad al encuentro
    basándonos en las señales AIS, VMS de la embarcación
-   Extraer los datos necesarios para poder valorar la calidad técnica de la señal
    GNSS del dispositivo móvil.


<a id="org1112e3d"></a>

## data

    data PeruvianVMSIdentifier =
      String as "P-04-00951"
    
    data ChileanVMSIdentifier =
      VesselName
    
    data VMSIdentifier =
      PeruvianVMSId
      OR ChileanVMSId
    
    data InformerDeviceLocation =
      RawGNSSMeasurementsDump
      OR BasicLocation
    
    data InformerAIS =
      MMSINumber
      OR WithoutOrPrivateAIS
    
    data InformerVMS =
      VMSIdentifier
      OR WithoutOrPrivateVMS
    
    data InformerData =
      InformerVMS
      AND InformerAIS
      AND InformerDeviceLocation


<a id="orge32162e"></a>

# Help Context


<a id="org949e85e"></a>

## Overview

El conocimiento de la legislación por los patrones o usuarios de la aplicación
es bajo. Igualmente lo referido a los sistemas de marcado o de radio
internacionales para flotas industriales.

Las especificaciones técnicas de las cámaras de los dispositivos móviles son
bajas, así como las oportunidades de obtener buenas tomas por distancia,
movimientos, suciedad de las embarcaciones&#x2026;


<a id="org2e7ebbf"></a>

### Goals

-   Educar a los usuarios sobre qué marcas del casco o del puente son importantes
    en la identifiación de un barco.
-   Informar de la localización mas común de las marcas.
-   Informar sobre los detalles constructivos que son mas sencillos de
    fotografiar.
-   Educar sobre las trasngresiones mas habituales en las flotas ilegales.

    data MarkingsHelp =
      Name
      AND Description
      AND list of Examples
      AND list of TypicalLocation
      AND list of ExamplePhotos
    
    data TypicalLocation =
      Stern
      OR Side
      OR Bridge


<a id="orgc876d40"></a>

# Scratch

Si hay una foto general del barco abre la puerta apoder incorporar fotos tomadas
con alguna cámara externa. Si no hay niguna es imposible verificar que
corresponda con el mismo encuentro.

No debo mezclar la definición del barco, los facts, con el estado en el momento
de la captura. Tratar de ocultar una marca, o estar realizando pesca ilegal, es
estado. Pero por ejemplo, tener callsign duplicados? es estado o hechos?


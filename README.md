
# Table of Contents

1.  [Overview](#orgcf9e81f)
2.  [Report Context](#org20af01f)
    1.  [Overview](#orgaf4a618)
    2.  [data](#orgcb17d5b)
    3.  [workflows](#orgd27f9a2)
3.  [Fishing Area Context](#org31823a7)
    1.  [Overview](#orgb41840c)
        1.  [Goals](#orgd83c969)
    2.  [data](#org68a4af0)
    3.  [workflows](#org3fe9d60)
        1.  [substeps](#org34b242e)
4.  [Vessel Identification Context](#orgc8a75f0)
    1.  [Overview](#org8a51648)
        1.  [Goals](#org38e21a4)
    2.  [Design trade-offs](#org038eb54)
    3.  [data](#org49de745)
5.  [Informer Data Context](#orge8c679f)
    1.  [Overview](#orgdb31efc)
        1.  [Goals](#org5a00720)
    2.  [data](#org02e5274)
6.  [Help Context](#orga451747)
    1.  [Overview](#org94f2c30)
        1.  [Goals](#orgc2c0f15)
    2.  [data](#org909c615)
7.  [Scratch](#orgc28c001)


<a id="orgcf9e81f"></a>

# Overview

![img](components-app.png)


<a id="org20af01f"></a>

# Report Context


<a id="orgaf4a618"></a>

## Overview

High sees encounters with alleged ilegal fishing vessels (mainly Asian flag
sheeps) are being reported by artisanal fishing fleet in Calamasur. They usually
don’t have the tools to record these encounters properly, and there is lack of
consensus on what information should be captured.

They usually take place in non network coverage áreas and may last up to 8 days

---

La flota pesquera artesanal denuncia que se producen encuentros en alta mar con
embarcaciones sospechosas de ejercer pesca ilegal. Principalmente flota de
bandera asiática. Carecen de herramientas específicas que les permitan registrar
estos encuentros y no existe un consenso sobre cuál es la información más
valiosa a capturar.

Estos encuentros se producen generalmente en zonas fuera de cobertura móvil y la
estancia en esas zonas puede durar hasta 8 días.


<a id="orgcb17d5b"></a>

## data

    data Report =
      FishingVessel
      AND NoEmptyList of SuspiciousActivity
      AND InformerData


<a id="orgd27f9a2"></a>

## workflows

    workflow "ReportSuspiciousActivity" =
      input = EmptyReport
      output (on success) = SuspiciousActivityReported
      output (on error) = InvalidReport
    
    //step 1
    do CheckFields


<a id="org31823a7"></a>

# Fishing Area Context


<a id="orgb41840c"></a>

## Overview

El proyecto va dirigido al ámbito de Calamasur. Los límites de las zonas
económicas exclusivas (EEZ) donde se producen el mayor número de encuentros
están definidos y aceptados internacionalmente.

Otras zonas de pesca tendrán problemas que serán total o sutilmente diferentes a
los de Calamasur, y sobre los que SFP no tiene actualmente acuerdos o competencias.
No es descartable que en futuras versiones se de soporte a estos otros problemas.

Todos los dispositivos son capaces de obtener su geolocalización.


<a id="orgd83c969"></a>

### Goals

-   Mejorar la experiencia de usuario con una interfaz y reglas de negocio
    adaptadas a la localización.
-   Comprobar que las capacidades de localización del dispositivo están encendidas.
-   Garantizar que al comienzo del proceso tenemos una posición minimamente
    precisa.
-   Minimizar el número de informes no válidos que moderar (Desde tierra, zonas
    sin competencias&#x2026;)


<a id="org68a4af0"></a>

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


<a id="org3fe9d60"></a>

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


<a id="org34b242e"></a>

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


<a id="orgc8a75f0"></a>

# Vessel Identification Context


<a id="org8a51648"></a>

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


<a id="org38e21a4"></a>

### Goals

-   Obtener la información mas relevante posible
-   Obtener avisos aun sin información relevante
-   Educar sobre los diferentes hechos denunciables, mas alla de la actividad
    pesquera.
-   Tratar de obtener pruebas gráficas, aun de calidad baja, donde se capturen
    detalles de la construcción de los buques.
-   Llegar a conocer si el sistema AIS está apagado o enviando datos falsos (spoofing).


<a id="org038eb54"></a>

## Design trade-offs

TODO: número de fotos permitidas
TODO: número de vídeos


<a id="org49de745"></a>

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


<a id="orge8c679f"></a>

# Informer Data Context


<a id="orgdb31efc"></a>

## Overview

El usuario o informador será generalmente un patrón de embarcación de pesca
artesanal. Suele ser reticente a desvelar su identidad o a hacer pública la
posición geográfica exacta del encuentro.

La flota artesanal carece de transmisores AIS, pero está en plena implantación
del sistema VMS. La flota industrial si transmite AIS públicamente.

Los dispositivos móviles Android que utilizan permiten obtener, no sólo la
posición, si no también volcados raw de la información de los sistemas de
navegación.


<a id="org5a00720"></a>

### Goals

-   Extraer el máximo contexto posible del momento de la captura
-   Tener la información necesaria para poder dar veracidad al encuentro
    basándonos en las señales AIS, VMS de la embarcación
-   Extraer los datos necesarios para poder valorar la calidad técnica de la señal
    GNSS del dispositivo móvil.


<a id="org02e5274"></a>

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


<a id="orga451747"></a>

# Help Context


<a id="org94f2c30"></a>

## Overview

El conocimiento de la legislación por los patrones o usuarios de la aplicación
es bajo. Igualmente lo referido a los sistemas de marcado o de radio
internacionales para flotas industriales.

Las especificaciones técnicas de las cámaras de los dispositivos móviles son
bajas, así como las oportunidades de obtener buenas tomas por distancia,
movimientos, suciedad de las embarcaciones&#x2026;


<a id="orgc2c0f15"></a>

### Goals

-   Educar a los usuarios sobre qué marcas del casco o del puente son importantes
    en la identifiación de un barco.
-   Informar de la localización mas común de las marcas.
-   Informar sobre los detalles constructivos que son mas sencillos de
    fotografiar.
-   Educar sobre las trasngresiones mas habituales en las flotas ilegales.


<a id="org909c615"></a>

## data

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


<a id="orgc28c001"></a>

# Scratch

Si hay una foto general del barco abre la puerta apoder incorporar fotos tomadas
con alguna cámara externa. Si no hay niguna es imposible verificar que
corresponda con el mismo encuentro.

No debo mezclar la definición del barco, los facts, con el estado en el momento
de la captura. Tratar de ocultar una marca, o estar realizando pesca ilegal, es
estado. Pero por ejemplo, tener callsign duplicados? es estado o hechos?


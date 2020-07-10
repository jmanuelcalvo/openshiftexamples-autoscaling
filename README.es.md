[![OpenShift Version][openshift39-logo]][openshift39-url]
[![OpenShift Version][openshift310-logo]][openshift310-url]
[![OpenShift Version][openshift311-logo]][openshift311-url]

# Ejemplos de OpenShift - Escalado automático
OpenShift puede escalar de forma manual o automática los pods de la aplicación hacia arriba y hacia abajo en función de las métricas del contenedor, como la CPU y el consumo de memoria. Este repositorio de git contiene un ejemplo intencionalmente simple de escalado automático (autoscaling) de una interfaz web con OpenShift.

Así es como se ve:

![Screenshot](./.screens/ocpautoscale.gif)

###### :information_source: Este ejemplo funciona en OpenShift Container Platform versión 3.11. Podría funcionar con otras versiones pero no ha sido probado

## ¿Cómo ejecutar esto?
En primer lugar, necesita acceso a un clúster OpenShift. ¿No tienes un clúster OpenShift? Está bien, descargue el CDK gratis aquí: [https://developers.redhat.com/products/cdk/overview/font>[5]. En segundo lugar, debe tener [métricas habilitadas en su clúster] [7].

Hay una plantilla para crear todos los componentes de este ejemplo. Use la herramienta oc CLI:

```
oc new-project autoscaledemo
oc new-app -f https://raw.githubusercontent.com/dudash/openshiftexamples-autoscaling/master/autoscale_instant_template.yaml
```

*Si no le gusta la CLI, otra opción es crear, proyectar e importar la plantilla a través de la consola web:*
  > Cree un nuevo proyecto, seleccione `Importar YAML/JSON` y luego cargue el archivo sin formato de este repositorio:` autoscale_instant_template.yaml` y asegúrese de que autoscaledemo esté configurado como el parámetro de espacio de nombres.

Ahora, para mostrar el escalado automático: simulemos una gran carga de usuarios en la aplicación web frontend utilizando Apache Benchmark. Si tiene `ab` instalado, ejecútelo contra la URL de la interfaz. O puede usar OpenShfit para extraer una [imagen del contenedor que contenga `ab`][6] y ejecutarla de forma automática como esta:

 > `oc run web-load --rm --attach --image=jordi/ab -- -n 50000 -c 10 http://URL_GOES_HERE/`

http://URL_GOES_HERE/ puede obtenerlo con el comando
 > `oc get route`

## ¿Por qué escalar automáticamente?
Dos de las principales razones para aprovechar esta capacidad en OpenShift:
1) Proporcione un mejor tiempo de actividad para sus aplicaciones cuando los picos de demanda del usuario
2) Reduzca el cálculo innecesario, reduzca la escala durante los períodos de inactividad y aumente cuando sea necesario

## ¿Cómo funciona esto y cómo puedo configurar el autoescalado?
Para definir la escala automática de una aplicación, primero definimos cuánta CPU y memoria debe consumir una instancia de la aplicación (min y max). Esto se convierte en la guía para que OpenShift sepa cuándo escalar el pod hacia arriba o hacia abajo. Estos detalles se colocan en lo que OpenShift llama una "configuración de implementación" ([puede leer más sobre eso aquí][1]).

En este ejemplo, si desea modificar algunas cosas en el ejemplo, las siguientes son las preguntas más comunes:

Cambie el número mínimo / máximo de contenedores replicados de la interfaz web y el objetivo de solicitud de CPU:
 > `oc autoscale dc/webapp --min 1 --max 5 --cpu-percent=60`

Cambie los valores de solicitud y límite para la implementación de la interfaz web:
 > `oc set resources dc/webapp --requests=cpu=200m,memory=256Mi --limits=cpu=400m,memory=512Mi`

Puede leer más sobre los detalles de la solicitud de recursos de cómputo y los límites [aquí][4]. Puede leer acerca de cómo se puede aprovechar la calidad de servicio (QoS) [aquí][3]. Y lea acerca de cómo configurar aún más la escala automática [aquí][2].


## Acerca del código / arquitectura de software
Las partes en acción aquí son:
* Interfaz web simple para recopilar la entrada del usuario y enviarla a una base de datos o capa API de back-end
* Servicio de backend, una base de datos MongoDB
* Archivo YAML de plantilla de aplicación instantánea (para crear / configurar todo fácilmente)
* Componentes clave de la plataforma que permiten este ejemplo
         * construcción de contenedores (fuente a imagen)
         * monitoreo de CPU / memoria de contenedores
         * replicación de contenedores y capa de servicio
         * Capa de red y enrutamiento definida por software

## License
Under the terms of the MIT.


[1]: https://docs.openshift.com/container-platform/3.7/architecture/core_concepts/deployments.html#deployments-and-deployment-configurations
[2]: https://docs.openshift.com/container-platform/3.7/dev_guide/pod_autoscaling.html
[3]: https://docs.openshift.com/container-platform/3.7/dev_guide/compute_resources.html#quality-of-service-tiers
[4]: https://docs.openshift.com/container-platform/3.7/dev_guide/compute_resources.html#dev-cpu-requests
[5]: https://developers.redhat.com/products/cdk/overview/
[6]: https://hub.docker.com/r/jordi/ab/
[7]: https://docs.openshift.com/container-platform/3.7/install_config/cluster_metrics.html

[openshift39-url]: https://docs.openshift.com/container-platform/3.9/welcome/index.html
[openshift310-url]: https://docs.openshift.com/container-platform/3.10/welcome/index.html
[openshift311-url]: https://docs.openshift.com/container-platform/3.11/welcome/index.html
[openshift41-url]: https://docs.openshift.com/container-platform/4.1/welcome/index.html

[openshift39-logo]: https://img.shields.io/badge/openshift-3.9-820000.svg?labelColor=grey&logo=data:image/png;base64,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

[openshift310-logo]: https://img.shields.io/badge/openshift-3.10-820000.svg?labelColor=grey&logo=data:image/png;base64,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

[openshift311-logo]: https://img.shields.io/badge/openshift-3.11-820000.svg?labelColor=grey&logo=data:image/png;base64,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

[openshift41-logo]: https://img.shields.io/badge/openshift-4.1-820000.svg?labelColor=grey&logo=data:image/png;base64,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

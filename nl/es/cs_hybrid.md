---

copyright:
  years: 2014, 2018
lastupdated: "2018-08-06"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}
{:tsSymptoms: .tsSymptoms}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}


# Utilización de {{site.data.keyword.containerlong_notm}} con {{site.data.keyword.Bluemix_notm}} privado
{: #hybrid_iks_icp}

Si tiene una cuenta privada de {{site.data.keyword.Bluemix}}, puede utilizarla con determinados servicios de {{site.data.keyword.Bluemix_notm}}, incluido {{site.data.keyword.containerlong}}. Para obtener más información, consulte el blog sobre [Experiencia híbrida en nube pública de IBM y privada de {{site.data.keyword.Bluemix_notm}}![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](http://ibm.biz/hybridJune2018).
{: shortdesc}

Conoce las [ofertas de {{site.data.keyword.Bluemix_notm}}](cs_why.html#differentiation). Ahora, puede [conectar su nube pública y privada ](#hybrid_vpn) y [ reutilizar los paquetes privados para contenedores públicos ](#hybrid_ppa_importer).

## Conexión de la nube pública y privada con la VPN de strongSwan
{: #hybrid_vpn}

Establezca conectividad de VPN entre el clúster de Kubernetes público y la instancia privada de {{site.data.keyword.Bluemix}} para permitir la comunicación bidireccional.
{: shortdesc}

1.  [Cree un clúster en {{site.data.keyword.Bluemix}} privado![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.3/installing/installing.html).

2.  Cree un clúster estándar con {{site.data.keyword.containerlong}} en {{site.data.keyword.Bluemix}} público o utilice uno existente. Para crear un clúster, elija entre las opciones siguientes: 
    - [Cree un clúster estándar desde la GUI ](cs_clusters.html#clusters_ui). 
    - [Cree un clúster estándar desde la CLI](cs_clusters.html#clusters_cli). 
    - [Utilice Cloud Automation Manager (CAM) para crear un clúster utilizando una plantilla predefinida![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://www.ibm.com/support/knowledgecenter/SS2L37_2.1.0.3/cam_deploy_IKS.html). Cuando se despliega un clúster con CAM, el tiller de Helm se instala automáticamente.

3.  En el clúster privado de {{site.data.keyword.Bluemix}}, despliegue el servicio VPN IPSec de strongSwan.

    1.  [Complete las soluciones alternativas de VPN IPSec de strongSwan ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") ](https://www.ibm.com/support/knowledgecenter/SS2L37_2.1.0.3/cam_strongswan.html). 

    2.  [Instale el diagrama de Helm de VPN de strongSwan![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.3/app_center/create_release.html) en su clúster privado.

4.  Obtenga la dirección IP pública de la pasarela VPN de {{site.data.keyword.Bluemix}} privado. La dirección IP forma parte de la [configuración preliminar del clúster privado![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.3/installing/prep_cluster.html).

5.  En el clúster de {{site.data.keyword.Bluemix}} público, [despliegue el servicio VPN de IPSec de strongSwan](cs_vpn.html#vpn-setup). Utilice la dirección IP pública del paso anterior y asegúrese de configurar la pasarela VPN en {{site.data.keyword.Bluemix}} público para la [conexión de salida](cs_vpn.html#strongswan_3) para que la conexión VPN se inicie desde el clúster en {{site.data.keyword.Bluemix}} público. 

6.  [Pruebe la conexión VPN](cs_vpn.html#vpn_test) entre los clústeres.

7.  Repita estos pasos para cada clúster que desee conectar. 


## Ejecución de imágenes de {{site.data.keyword.Bluemix_notm}} privado en contenedores públicos de Kubernetes
{: #hybrid_ppa_importer}

Puede ejecutar determinados productos bajo licencia de IBM empaquetados para {{site.data.keyword.Bluemix_notm}} privado en un clúster de {{site.data.keyword.Bluemix_notm}} público.  
{: shortdesc}

El software bajo licencia está disponible en [IBM Passport Advantage ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://www-01.ibm.com/software/passportadvantage/index.html). Para utilizar este software en un clúster de {{site.data.keyword.Bluemix_notm}} público, debe descargar el software, extraer la imagen y cargar la imagen en el espacio de nombres en {{site.data.keyword.registryshort}}. Independientemente del entorno en el que tenga previsto utilizar el software, debe obtener primero la licencia necesaria para el producto. 

En la tabla siguiente se ofrece una visión general de los productos disponibles de {{site.data.keyword.Bluemix_notm}} privado que puede utilizar en el clúster de {{site.data.keyword.Bluemix_notm}} público.

| Nombre del producto | Versión | Número de pieza |
| --- | --- | --- |
| IBM Db2 Direct Advanced Edition Server | 11.1 | CNU3TML |
| IBM Db2 Advanced Enterprise Server Edition Server | 11.1 | CNU3SML |
| IBM MQ Advanced | 9.0.5 | CNU1VML |
| IBM WebSphere Application Server Liberty | 16.0.0.3 | Imagen de Docker Hub |
{: caption="Tabla. Productos de {{site.data.keyword.Bluemix_notm}} privado soportados que se pueden utilizar en {{site.data.keyword.Bluemix_notm}} público." caption-side="top"}

Antes de empezar: 
- [Instale el plugin de la CLI de {{site.data.keyword.registryshort}} (`ibmcloud cr`)](/docs/services/Registry/registry_setup_cli_namespace.html#registry_cli_install). 
- [Configure un espacio de nombres en {{site.data.keyword.registryshort}}](/docs/services/Registry/registry_setup_cli_namespace.html#registry_namespace_add) o recupere el espacio de nombres existente ejecutando `ibmcloud cr namespaces`. 
- [Defina su clúster como destino de la CLI de `kubectl`](/docs/containers/cs_cli_install.html#cs_cli_configure). 
- [Instale la CLI de Helm y configure el servicio tiller en el clúster](/docs/containers/cs_integrations.html#helm). 

Para desplegar una imagen de {{site.data.keyword.Bluemix_notm}} privado en {{site.data.keyword.Bluemix_notm}} público:

1.  Siga los pasos de la [documentación de {{site.data.keyword.registryshort}}](/docs/services/Registry/ts_index.html#ts_ppa) para descargar el software bajo licencia de IBM Passport Advantage, enviar por push la imagen al espacio de nombres e instalar el diagrama de Helm en el clúster. 

    **Para IBM WebSphere Application Server Liberty**:
    
    1.  En lugar de obtener la imagen de IBM Passport Advantage, utilice la [imagen de Docker Hub ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://hub.docker.com/_/websphere-liberty/). Para obtener instrucciones sobre cómo obtener una licencia de producción, consulte [Actualización de la imagen de Docker Hub a una imagen de producción![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://github.com/WASdev/ci.docker/tree/master/ga/production-upgrade).
    
    2.  Siga las [instrucciones
del diagrama de Helm de Liberty![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_icp_helm.html). 

2.  Verifique que el campo **STATUS** del diagrama de Helm muestra `DEPLOYED`. Si no es así, espere unos minutos e inténtelo de nuevo.
    ```
    helm status <helm_chart_name>
    ```
    {: pre}
   
3.  Consulte la documentación específica del producto para obtener más información sobre cómo configurar y utilizar el producto con el clúster. 

    - [IBM Db2 Direct Advanced Edition Server ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.licensing.doc/doc/c0070181.html) 
    - [IBM MQ Advanced ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.0.0/com.ibm.mq.helphome.v90.doc/WelcomePagev9r0.html)
    - [IBM WebSphere Application Server Liberty ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/as_ditamaps/was900_welcome_liberty.html)

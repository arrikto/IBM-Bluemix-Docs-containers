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


# 클러스터의 서브넷 구성
{: #subnets}

Kubernetes 클러스터에 서브넷을 추가하여 로드 밸런서 서비스에 사용 가능한 포터블 공인 또는 사설 IP 주소의 풀을 변경하십시오.
{:shortdesc}

## 클러스터의 기본 VLAN, 서브넷 및 IP
{: #default_vlans_subnets}

클러스터 작성 중에 클러스터 작업자 노드 및 기본 서브넷은 자동으로 VLAN에 연결됩니다. 

### VLAN
{: #vlans}

클러스터를 작성하면 클러스터의 작업자 노드가 VLAN에 자동으로 연결됩니다. VLAN은 동일한 실제 회선에 연결된 것처럼 작업자 노드 및 팟(Pod)의 그룹을 구성하며, 작업자 및 팟(Pod) 간의 연결을 위한 채널을 제공합니다. 

<dl>
<dt>무료 클러스터의 VLAN</dt>
<dd>무료 클러스터의 경우에는 기본적으로 클러스터의 작업자 노드가 IBM 소유의 공용 VLAN 및 사설 VLAN에 연결됩니다. IBM에서 VLAN, 서브넷 및 IP 주소를 제어하므로 사용자는 다중 구역 클러스터를 작성하거나 클러스터에 서브넷을 추가할 수 없습니다. 오직 NodePort 서비스를 사용한 앱 노출만 수행할 수 있습니다. </dd>
<dt>표준 클러스터의 VLAN</dt>
<dd>표준 클러스터에서 구역의 클러스터를 처음으로 작성하는 경우, 해당 구역의 공용 VLAN 및 사설 VLAN은 IBM Cloud 인프라(SoftLayer) 계정에서 사용자를 위해 자동으로 프로비저닝됩니다.  다중 클러스터가 VLAN을 공유할 수 있으므로, 해당 구역에서 작성된 모든 후속 클러스터에 대해 동일한 공용 및 사설 VLAN을 재사용할 수 있습니다. </br></br>작업자 노드를 공용 VLAN 및 사설 VLAN 모두에 연결하거나 사설 VLAN에만 연결할 수 있습니다. 작업자 노드를 사설 VLAN에만 연결하려는 경우에는 기존 사설 VLAN의 ID를 사용하거나 [새 사설 VLAN을 작성](/docs/cli/reference/softlayer/index.html#sl_vlan_create)하여 클러스터 작성 중에 해당 ID를 사용할 수 있습니다.</dd></dl>

계정에 대해 각 구역에서 프로비저닝되는 VLAN을 보려면 `ibmcloud ks vlans <zone>`을 실행하십시오. 하나의 클러스터가 프로비저닝된 VLAN을 보려면 `ibmcloud ks cluster-get <cluster_name_or_ID> --showResources`를 실행하고 **Subnet VLANs** 섹션을 찾으십시오. 

**참고**: 다중 구역 클러스터, 단일 구역 클러스터용 다중 VLAN 또는 동일한 VLAN의 다중 서브넷이 있는 경우에는 작업자 노드가 사설 네트워크에서 서로 간에 통신할 수 있도록 VLAN Spanning을 켜야 합니다. 지시사항은 [VLAN Spanning 사용 또는 사용 안함](/docs/infrastructure/vlans/vlan-spanning.html#vlan-spanning)을 참조하십시오.

### 서브넷 및 IP 주소
{: #subnets_ips}

작업자 노드 및 팟(Pod)에 추가하여 서브넷도 자동으로 VLAN에 프로비저닝됩니다. 서브넷은 클러스터 컴포넌트에 IP 주소를 지정하여 이에 대한 네트워크 연결을 제공합니다. 

다음의 서브넷이 기본 공용 및 사설 VLAN에서 자동으로 프로비저닝됩니다. 

**공용 VLAN 서브넷**
* 기본 공용 서브넷은 클러스터 작성 중에 작업자 노드에 지정된 공인 IP 주소를 판별합니다. 동일한 VLAN의 다중 클러스터는 하나의 기본 공용 서브넷을 공유할 수 있습니다. 
* 포터블 공용 서브넷은 하나의 클러스터에만 바인드되어 있으며 8개의 공인 IP 주소를 클러스터에 제공합니다. 3개의 IP는 Softlayer 기능용으로 예약되어 있습니다. 1개의 IP는 기본 공용 Ingress ALB에 의해 사용되며, 4개의 IP는 공용 로드 밸런서 네트워킹 서비스 작성에 사용될 수 있습니다. 포터블 공인 IP는 인터넷을 통해 로드 밸런서 서비스에 액세스하기 위해 사용될 수 있는 영구적인 고정 IP 주소입니다. 공용 로드 밸런서에 대해 5개 이상의 IP가 필요하면 [포터블 IP 주소 추가](#adding_ips)를 참조하십시오. 

**사설 VLAN 서브넷**
* 기본 사설 서브넷은 클러스터 작성 중에 작업자 노드에 지정된 사설 IP 주소를 판별합니다. 동일한 VLAN의 다중 클러스터는 하나의 기본 사설 서브넷을 공유할 수 있습니다. 
* 포터블 사설 서브넷은 하나의 클러스터에만 바인드되어 있으며 8개의 사설 IP 주소를 클러스터에 제공합니다. 3개의 IP는 Softlayer 기능용으로 예약되어 있습니다. 1개의 IP는 기본 사설 Ingress ALB에 의해 사용되며, 4개의 IP는 사설 로드 밸런서 네트워킹 서비스 작성에 사용될 수 있습니다. 포터블 사설 IP는 인터넷을 통해 로드 밸런서 서비스에 액세스하기 위해 사용될 수 있는 영구적인 고정 IP 주소입니다. 사설 로드 밸런서에 대해 5개 이상의 IP가 필요하면 [포터블 IP 주소 추가](#adding_ips)를 참조하십시오. 

계정에서 프로비저닝된 서브넷을 모두 보려면 `ibmcloud ks subnets`를 실행하십시오. 하나의 클러스터에 바인드된 포터블 공인 및 포터블 사설 서브넷을 보려면 `ibmcloud ks cluster-get <cluster_name_or_ID> --showResources`를 실행하고 **Subnet VLANs** 섹션을 찾으십시오. 

**참고**: {{site.data.keyword.containershort_notm}}에서 VLAN의 서브넷 한계는 40개입니다. 이 한계에 도달하면 우선 [VLAN의 서브넷을 재사용하여 새 클러스터를 작성](#custom)할 수 있는지 여부를 확인하십시오. 새 VLAN이 필요하면 [{{site.data.keyword.Bluemix_notm}} 지원에 문의](/docs/infrastructure/vlans/order-vlan.html#order-vlans)하여 VLAN을 주문하십시오. 그런 다음, 이 새 VLAN을 사용하는 [클러스터를 작성](cs_cli_reference.html#cs_cluster_create)하십시오.

<br />


## 사용자 정의 또는 기존 서브넷을 사용하여 클러스터 작성
{: #custom}

표준 클러스터를 작성하면 서브넷이 자동으로 작성됩니다. 그러나 자동으로 프로비저닝된 서브넷을 사용하는 대신에 IBM Cloud 인프라(SoftLayer) 계정에서 기존의 포터블 서브넷을 사용하거나 삭제된 클러스터의 서브넷을 재사용할 수 있습니다.
{:shortdesc}

클러스터 제거 및 작성 간에 안정된 정적 IP 주소를 유지하거나 더 큰 IP 주소 블록을 주문하려면 이 옵션을 사용하십시오.

**참고:** 포터블 공인 IP 주소는 월별로 비용이 청구됩니다. 클러스터가 프로비저닝된 후 포터블 공인 IP 주소를 제거한 경우, 짧은 시간 동안만 사용한 경우에도 월별 비용은 여전히 지불해야 합니다.

시작하기 전에 다음 작업을 수행하십시오.
- 클러스터에 [CLI를 대상으로 지정](cs_cli_install.html#cs_cli_configure)하십시오.
- 더 이상 필요하지 않은 클러스터의 서브넷을 재사용하려면 필요하지 않은 클러스터를 삭제하십시오. 재사용하지 않으면 24시간 내에 서브넷이 삭제되므로, 새 클러스터를 즉시 작성하십시오. 

   ```
   ibmcloud ks cluster-rm <cluster_name_or_ID>
   ```
   {: pre}

사용자 정의 방화벽 규칙 또는 사용 가능한 IP 주소를 사용하여 IBM Cloud 인프라(SoftLayer) 포트폴리오의 기존 서브넷을 사용하려면 다음을 수행하십시오.

1. 사용하고자 하는 서브넷의 ID와 서브넷이 위치한 VLAN의 ID를 가져오십시오. 

    ```
    ibmcloud ks subnets
    ```
    {: pre}

    이 예제 출력에서 서브넷 ID는 `1602829`이며 VLAN ID는 `2234945`입니다.
    ```
        Getting subnet list...
    OK
    ID        Network             Gateway          VLAN ID   Type      Bound Cluster
    1550165   10.xxx.xx.xxx/26    10.xxx.xx.xxx    2234947   private
    1602829   169.xx.xxx.xxx/28   169.xx.xxx.xxx   2234945   public
    ```
    {: screen}

2. 식별된 VLAN ID를 사용하여 [클러스터를 작성](cs_clusters.html#clusters_cli)하십시오. 새 포터블 공인 IP 서브넷 및 새 포터블 사설 IP 서브넷이 자동으로 작성되지 않도록 `--no-subnet` 플래그를 포함하십시오.

    ```
    ibmcloud ks cluster-create --zone dal10 --machine-type b2c.4x16 --no-subnet --public-vlan 2234945 --private-vlan 2234947 --workers 3 --name my_cluster
    ```
    {: pre}
    `--zone` 플래그에 대해 VLAN이 있는 구역을 기억할 수 없는 경우에는 `ibmcloud ks vlans <zone>`을 실행하여 VLAN이 특정 구역에 있는지 여부를 확인할 수 있습니다.
    {: tip}

3.  클러스터가 작성되었는지 확인하십시오. **참고:** 작업자 노드 시스템을 정렬하고 클러스터를 설정하며 계정에서 프로비저닝하는 데는 최대 15분 정도 소요될 수 있습니다. 

    ```
    ibmcloud ks clusters
    ```
    {: pre}

    클러스터가 완전히 프로비저닝되면 **상태(State)**가 `deployed`로 변경됩니다. 

    ```
    Name         ID                                   State      Created          Workers   Zone   Version
    mycluster    aaf97a8843a29941b49a598f516da72101   deployed   20170201162433   3         dal10      1.10.5
    ```
    {: screen}

4.  작업자 노드의 상태를 확인하십시오.

    ```
    ibmcloud ks workers <cluster_name_or_ID>
    ```
    {: pre}

    다음 단계를 계속하려면 우선 작업자 노드가 준비되어 있어야 합니다. **상태(State)**가 `normal`로 변경되며 **상태(Status)**는 `Ready`입니다. 

    ```
    ID                                                  Public IP        Private IP     Machine Type   State      Status   Zone   Version
    prod-dal10-pa8dfcc5223804439c87489886dbbc9c07-w1    169.xx.xxx.xxx   10.xxx.xx.xxx  free           normal     Ready    dal10      1.10.5
    ```
    {: screen}

5.  서브넷 ID를 지정하여 클러스터에 서브넷을 추가하십시오. 클러스터가 서브넷을 사용할 수 있도록 하는 경우에는 사용자가 사용할 수 있는 모든 사용 가능한 포터블 공인 IP 주소가 포함된 Kubernetes configmap이 사용자를 위해 작성됩니다. 서브넷의 VLAN이 있는 구역에 Ingress ALB가 존재하지 않는 경우에는 하나의 포터블 공인 및 하나의 포터블 사설 IP 주소를 자동으로 사용하여 해당 구역에 대한 공용 및 사설 ALB를 작성합니다. 사용자는 서브넷의 기타 모든 포터블 공인 및 사설 IP 주소를 사용하여 앱에 대한 로드 밸런서 서비스를 작성할 수 있습니다. 

    ```
    ibmcloud ks cluster-subnet-add mycluster 807861
    ```
    {: pre}

6. **중요**: 동일한 VLAN의 서로 다른 서브넷에 있는 작업자 간에 통신을 사용 가능하게 하려면 [동일한 VLAN의 서브넷 간에 라우팅 사용](#subnet-routing)을 설정해야 합니다. 

<br />


## 기존 포터블 IP 주소 관리
{: #managing_ips}

기본적으로, 4개의 포터블 공인 및 4개의 포터블 사설 IP 주소는 [로드 밸런서 서비스 작성](cs_loadbalancer.html)으로 단일 앱을 공용 또는 사설 네트워크에 노출시키는 데 사용될 수 있습니다. 로드 밸런서 서비스를 작성하려면 사용 가능한 올바른 유형의 포터블 IP 주소가 1개 이상 있어야 합니다. 사용 가능한 포터블 IP 주소를 보거나 사용된 포터블 IP 주소를 해제할 수 있습니다. 

### 사용 가능한 포터블 공인 IP 주소 보기
{: #review_ip}

사용되는 클러스터와 사용 가능한 클러스터 모두의 모든 포터블 IP 주소를 나열하려면 다음을 실행할 수 있습니다. 

```
kubectl get cm ibm-cloud-provider-vlan-ip-config -n kube-system -o yaml
```
{: pre}

로드 밸런서를 작성하는 데 사용할 수 있는 포터블 공인 IP 주소만 나열하려면 다음 단계를 사용할 수 있습니다. 

시작하기 전에, [사용할 클러스터의 컨텍스트를 설정하십시오.](cs_cli_install.html#cs_cli_configure)

1.  `myservice.yaml`이라는 이름의 Kubernetes 서비스 구성 파일을 작성하고 더미 로드 밸런서 IP 주소를 사용하여 `LoadBalancer` 유형의 서비스를 정의하십시오. 다음 예는 로드 밸런서 IP 주소로 IP 주소 1.1.1.1을 사용합니다.

    ```
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        run: myservice
      name: myservice
      namespace: default
    spec:
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        run: myservice
      sessionAffinity: None
      type: LoadBalancer
      loadBalancerIP: 1.1.1.1
    ```
    {: codeblock}

2.  클러스터에 서비스를 작성하십시오.

    ```
    kubectl apply -f myservice.yaml
    ```
    {: pre}

3.  서비스를 검사하십시오.

    ```
    kubectl describe service myservice
    ```
    {: pre}

    **참고:** Kubernetes 마스터가 Kubernetes configmap에서 지정된 로드 밸런서 IP 주소를 찾을 수 없기 때문에 이 서비스의 작성에 실패합니다. 이 명령을 실행하면 오류 메시지 및 클러스터에 사용 가능한 공인 IP 주소의 목록을 볼 수 있습니다.

    ```
    Error on cloud load balancer a8bfa26552e8511e7bee4324285f6a4a for service default/myservice with UID 8bfa2655-2e85-11e7-bee4-324285f6a4af: Requested cloud provider IP 1.1.1.1 is not available. The following cloud provider IP addresses are available: <list_of_IP_addresses>
    ```
    {: screen}

<br />


### 사용된 IP 주소 해제
{: #free}

포터블 IP 주소를 사용 중인 로드 밸런서 서비스를 삭제하여 사용된 포터블 공인 IP 주소를 해제할 수 있습니다.
{:shortdesc}

시작하기 전에, [사용할 클러스터의 컨텍스트를 설정하십시오.](cs_cli_install.html#cs_cli_configure)

1.  클러스터에서 사용 가능한 서비스를 나열하십시오.

    ```
     kubectl get services
    ```
    {: pre}

2.  공인 또는 사설 IP 주소를 사용하는 로드 밸런서 서비스를 제거하십시오.

    ```
    kubectl delete service <service_name>
    ```
    {: pre}

<br />


## 포터블 IP 주소 추가
{: #adding_ips}

기본적으로, 4개의 포터블 공인 및 4개의 포터블 사설 IP 주소는 [로드 밸런서 서비스 작성](cs_loadbalancer.html)으로 단일 앱을 공용 또는 사설 네트워크에 노출시키는 데 사용될 수 있습니다. 4개 이상의 공용 또는 4개 이상의 사설 로드 밸런서를 작성하기 위해 클러스터에 네트워크 서브넷을 추가하여 더 많은 포터블 IP 주소를 가져올 수 있습니다. 

**참고:**
* 클러스터에 서브넷을 사용 가능하게 하면 이 서브넷의 IP 주소가 클러스터 네트워킹 목적으로 사용됩니다. IP 주소 충돌을 피하려면 하나의 클러스터만 있는 서브넷을 사용해야 합니다. 동시에
{{site.data.keyword.containershort_notm}}의 외부에서
다른 목적으로 또는 다중 클러스터에 대한 서브넷으로 사용하지 마십시오.
* 포터블 공인 IP 주소는 매월 비용이 청구됩니다. 서브넷이 프로비저닝된 후에 포터블 공인 IP 주소를 제거한 경우에는 짧은 시간 동안만 사용했어도 여전히 월별 비용을 지불해야 합니다. 

### 추가로 서브넷을 주문하여 포터블 IP 추가
{: #request}

IBM Cloud 인프라(SoftLayer) 계정에서 새 서브넷을 작성하고 지정된 클러스터에서 이를 사용할 수 있도록 하여 로드 밸런서 서비스를 위한 포터블 IP를 더 가져올 수 있습니다.
{:shortdesc}

시작하기 전에 클러스터에 [CLI를 대상으로 지정](cs_cli_install.html#cs_cli_configure)하십시오.

1. 새 서브넷을 프로비저닝하십시오.

    ```
    ibmcloud ks cluster-subnet-create <cluster_name_or_id> <subnet_size> <VLAN_ID>
    ```
    {: pre}

    <table>
    <caption>이 명령의 컴포넌트 이해</caption>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="아이디어 아이콘"/> 이 명령의 컴포넌트 이해</th>
    </thead>
    <tbody>
    <tr>
    <td><code>cluster-subnet-create</code></td>
    <td>클러스터를 위한 서브넷을 프로비저닝하는 명령입니다.</td>
    </tr>
    <tr>
    <td><code><em>&lt;cluster_name_or_id&gt;</em></code></td>
    <td><code>&lt;cluster_name_or_id&gt;</code>를 클러스터의 이름 또는 ID로 대체하십시오.</td>
    </tr>
    <tr>
    <td><code><em>&lt;subnet_size&gt;</em></code></td>
    <td><code>&lt;subnet_size&gt;</code>를 포터블 서브넷에서 추가할 IP 주소의 수로 대체하십시오. 허용되는 값은 8, 16, 32 또는 64입니다. <p>**참고:** 서브넷에 대한 포터블 IP 주소를 추가할 때 세 개의 IP 주소를 사용하여 클러스터 내부 네트워킹을 설정합니다. 애플리케이션 로드 밸런서에 대해 또는 로드 밸런서 서비스를 작성하는 데 세 개의 IP 주소를 사용할 수 없습니다. 예를 들어, 8개의 포터블 공인 IP 주소를 요청하는 경우 이 중에서 5개를 사용하여 앱을 공용으로 노출할 수 있습니다.</p> </td>
    </tr>
    <tr>
    <td><code><em>&lt;VLAN_ID&gt;</em></code></td>
    <td><code>&lt;VLAN_ID&gt;</code>를 포터블 공인 또는 사설 IP 주소를 할당할 공용 또는 사설 VLAN의 ID로 대체하십시오. 기존 작업자 노드가 연결되어 있는 공용 또는 사설 VLAN을 선택해야 합니다. 작업자 노드에 대한 공용 또는 사설 VLAN을 검토하려면 <code>ibmcloud ks worker-get &lt;worker_id&gt;</code> 명령을 실행하십시오. <서브넷은 VLAN이 있는 동일 구역에서 프로비저닝됩니다. </td>
    </tr>
    </tbody></table>

2. 서브넷이 정상적으로 작성되어 클러스터에 추가되었는지 확인하십시오. 서브넷 CIDR은 **Subnet VLANs** 섹션에 나열되어 있습니다. 

    ```
    ibmcloud ks cluster-get --showResources <cluster_name_or_ID>
    ```
    {: pre}

    이 예제 출력에서는 두 번째 서브넷이 `2234945` 공용 VLAN에 추가되었습니다.
    ```
    Subnet VLANs
    VLAN ID   Subnet CIDR          Public   User-managed
    2234947   10.xxx.xx.xxx/29     false    false
    2234945   169.xx.xxx.xxx/29    true     false
    2234945   169.xx.xxx.xxx/29    true     false
    ```
    {: screen}

3. **중요**: 동일한 VLAN의 서로 다른 서브넷에 있는 작업자 간에 통신을 사용 가능하게 하려면 [동일한 VLAN의 서브넷 간에 라우팅 사용](#subnet-routing)을 설정해야 합니다. 

<br />


### 사용자 관리 서브넷을 사용하여 포터블 사설 IP 추가
{: #user_managed}

지정된 클러스터에 사용 가능한 온프레미스 네트워크에서 서브넷을 작성하여 로드 밸런서 서비스에 대한 추가 포터블 사설 IP를 가져올 수 있습니다.
{:shortdesc}

요구사항:
- 사용자가 관리하는 서브넷은 사설 VLAN에만 추가할 수 있습니다.
- 서브넷 접두부 길이 한계는 /24 - /30입니다. 예를 들면, `169.xx.xxx.xxx/24`는 253개의 사용 가능한 사설 IP 주소를 지정하지만 `169.xx.xxx.xxx/30`은 1개의 사용 가능한 사설 IP 주소를 지정합니다.
- 서브넷의 첫 번째 IP 주소는 서브넷에 대한 게이트웨이로 사용되어야 합니다.

시작하기 전에:
- 외부 서브넷에 들어오고 나가는 네트워크 트래픽의 라우팅을 구성하십시오.
- 온프레미스 데이터센터 게이트웨이 디바이스와 사설 네트워크 가상 라우터 어플라이언스 또는 클러스터에서 실행되는 ngSwan VPN 서비스 간의 VPN 연결이 있는지 확인하십시오. 자세한 정보는 [VPN 연결 설정](cs_vpn.html)을 참조하십시오.

온프레미스 네트워크의 서브넷을 추가하려면 다음을 수행하십시오.

1. 클러스터의 사설 VLAN의 ID를 보십시오. **Subnet VLANs** 섹션을 찾으십시오. **User-managed** 필드에서 _false_인 VLAN ID를 식별하십시오.

    ```
    ibmcloud ks cluster-get --showResources <cluster_name>
    ```
    {: pre}

    ```
Subnet VLANs
  VLAN ID   Subnet CIDR         Public   User-managed
  2234947   10.xxx.xx.xxx/29    false    false
  2234945   169.xx.xxx.xxx/29  true    false
    ```
    {: screen}

2. 사설 VLAN에 외부 서브넷을 추가하십시오. 포터블 사설 IP 주소가 클러스터의 configmap에 추가됩니다.

    ```
    ibmcloud ks cluster-user-subnet-add <cluster_name> <subnet_CIDR> <VLAN_ID>
    ```
    {: pre}

    예:

    ```
    ibmcloud ks cluster-user-subnet-add mycluster 10.xxx.xx.xxx/24 2234947
    ```
    {: pre}

3. 사용자 제공 서브넷이 추가되었는지 확인하십시오. **User-managed** 필드가 _true_입니다.

    ```
    ibmcloud ks cluster-get --showResources <cluster_name>
    ```
    {: pre}

    이 예제 출력에서는 두 번째 서브넷이 `2234947` 사설 VLAN에 추가되었습니다.
    ```
    Subnet VLANs
    VLAN ID   Subnet CIDR       Public   User-managed
    2234947   10.xxx.xx.xxx/29  false    false
    2234945   169.xx.xxx.xxx/29 true     false
    2234947   10.xxx.xx.xxx/24  false    true
    ```
    {: screen}

4. [동일한 VLAN의 서브넷 간에 라우팅 사용](#subnet-routing)을 수행하십시오.

5. [사설 로드 밸런서 서비스](cs_loadbalancer.html)를 추가하거나 [사설 Ingress ALB](cs_ingress.html#private_ingress)를 사용하여 사설 네트워크를 통해 앱에 액세스하십시오. 사용자가 추가한 서브넷의 사설 IP 주소를 사용하려면 서브넷 CIDR의 IP 주소를 지정해야 합니다. 그렇지 않으면 IBM Cloud 인프라(SoftLayer) 서브넷 또는 사설 VLAN의 사용자 제공 서브넷에서 랜덤으로 IP 주소가 선택됩니다.

<br />


## 서브넷 라우팅 관리
{: #subnet-routing}

다중 구역 클러스터, 단일 구역 클러스터용 다중 VLAN 또는 동일한 VLAN의 다중 서브넷이 있는 경우에는 작업자 노드가 사설 네트워크에서 서로 간에 통신할 수 있도록 VLAN Spanning을 켜야 합니다. 지시사항은 [VLAN Spanning 사용 또는 사용 안함](/docs/infrastructure/vlans/vlan-spanning.html#vlan-spanning)을 참조하십시오.

VLAN Spanning이 역시 필요한 다음의 시나리오를 검토하십시오. 

### 동일한 VLAN의 기본 서브넷 간에 라우팅 사용
{: #vlan-spanning}

클러스터를 작성하는 경우에는 `/26`에서 끝나는 서브넷이 기본 사설 기본 VLAN에서 프로비저닝됩니다. 사설 기본 서브넷은 최대 62개의 작업자 노드에 대한 IP를 제공할 수 있습니다.
{:shortdesc}

이 62개의 작업자 노드 한계는 동일한 VLAN에 있는 단일 지역의 대형 클러스터 또는 여러 소형 클러스터에서 초과될 수 있습니다. 62개의 작업자 노드 한계에 도달하는 경우에는 동일한 VLAN의 두 번째 사설 기본 서브넷이 주문됩니다. 

동일한 VLAN의 이러한 기본 서브넷에서 작업자가 통신할 수 있도록 보장하려면 VLAN Spanning을 켜야 합니다. 지시사항은 [VLAN Spanning 사용 또는 사용 안함](/docs/infrastructure/vlans/vlan-spanning.html#vlan-spanning)을 참조하십시오.

### 게이트웨이 어플라이언스에 대한 서브넷 라우팅 관리
{: #vra-routing}

표준 클러스터를 작성하면 클러스터가 연결된 VLAN에서 포터블 공인 및 포터블 사설 서브넷이 주문됩니다. 이러한 서브넷은 Ingress 및 로드 밸런서 네트워킹 서비스에 대한 IP 주소를 제공합니다. 

그러나 [VRA(Virtual Router Appliance)](/docs/infrastructure/virtual-router-appliance/about.html#about) 등의 기존 라우터 어플라이언스가 있는 경우에는 클러스터가 연결된 해당 VLAN의 새로 추가된 포터블 서브넷이 라우터에 구성되어 있지 않습니다. Ingress 또는 로드 밸런서 네트워킹 서비스를 사용하려면 [VLAN Spanning을 사용으로 설정](/docs/infrastructure/vlans/vlan-spanning.html#vlan-spanning)하여 네트워크 디바이스가 동일한 VLAN의 서로 다른 서브넷 간에 라우팅할 수 있도록 보장해야 합니다. 

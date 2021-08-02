#### Create Ingress Gateway to include Redis Enterprise Database instances

Define gateway for TCP access:
```
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: redis-gateway
spec:
  selector:
    istio: ingressgateway # use istio default ingress gateway
  servers:
  - port:
      number: ${SECURE_INGRESS_PORT}
      name: https
      protocol: HTTPS
    tls:
      mode: PASSTHROUGH
    hosts:
    - rec-ui.${INGRESS_HOST}.nip.io
  - port:
      number: ${DB_PORT}
      name: redis-${DB_PORT}
      protocol: TCP
    hosts:
    - redis-${DB_PORT}.demo.rec.${INGRESS_HOST}.nip.io
  - port:
      number: ${DB_PORT_2}
      name: redis-${DB_PORT_2}
      protocol: TCP
    hosts:
    - redis-${DB_PORT_2}.demo.rec.${INGRESS_HOST}.nip.io

EOF
```  
  
Configure routes for traffic entering via the gateway for the databases:
```
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: redis-bdbs
spec:
  hosts:
  - "*.demo.rec.${INGRESS_HOST}.nip.io"
  gateways:
  - redis-gateway
  tcp:
  - match:
    - port: ${DB_PORT}
    route:
    - destination:
        host: redis-enterprise-database
        port:
          number: ${DB_PORT}
  - match:
    - port: ${DB_PORT_2}
    route:
    - destination:
        host: redis-enterprise-database-two
        port:
          number: ${DB_PORT_2}
EOF
```  
  
Add custom ports for Redis Enterprise database connections to default ingress gateway
```
kubectl edit svc istio-ingressgateway -n istio-system
```
Add the following next to other port definitions:
```
- name: redis-port
  nodePort: <node-port-of-your-choice>
  port: <replace with ${DB_PORT}>
  protocol: TCP
  targetPort: <replace with ${DB_PORT}>
- name: redis-port
  nodePort: <node-port-of-your-choice>
  port: <replace with ${DB_PORT_2}>
  protocol: TCP
  targetPort: <replace with ${DB_PORT_2}>

For example,
- name: redis-port
  nodePort: 31402
  port: 13813
  protocol: TCP
  targetPort: 13813
- name: redis-port-2
  nodePort: 31403
  port: 11886
  protocol: TCP
  targetPort: 11886
```
  
  

---

apiVersion: v1
kind: Service
metadata:
  name: {{ app_name }}
  namespace: {{ namespace }}
spec:
  type: NodePort
  ports:
  - port: {{ listen_port }}
    targetPort: {{ listen_port }}
    protocol: TCP
  selector:
    app: {{ app_name }}

{% if aws_loadbalancer_controller_version == "v1" %}
---

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: "{{ app_name }}"
  namespace: {{ namespace }}
  annotations:
    kubernetes.io/ingress.class: alb
    # specifies whether your LoadBalancer will be internet facing
    alb.ingress.kubernetes.io/scheme: internet-facing
    # pecifies how to route traffic to pods. You can choose between instance and ip
    alb.ingress.kubernetes.io/target-type: ip
    # specifies the ports that ALB used to listen on
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
    # specifies the Security Policy that should be assigned to the ALB, allowing you to control the protocol and ciphers.
    alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-TLS-1-1-2017-01
    # specifies the ARN of one or more certificate managed by AWS Certificate Manager
    alb.ingress.kubernetes.io/certificate-arn: {{ alb_ssl_certificate_arn }}
    # specifies Load Balancer Attributes that should be applied to the ALB
    alb.ingress.kubernetes.io/load-balancer-attributes: idle_timeout.timeout_seconds=600,routing.http2.enabled=true
    # specifies the consecutive health checks successes required before considering an unhealthy target healthy.
    alb.ingress.kubernetes.io/healthy-threshold-count: '2'
    # specifies the consecutive health check failures required before considering a target unhealthy
    alb.ingress.kubernetes.io/unhealthy-threshold-count: '3'
    # specifies the HTTP path when peforming health check on targets.
    alb.ingress.kubernetes.io/healthcheck-path: {{ liveness_probe_uri }}
    # specifies the interval(in seconds) between health check of an individual target
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: '10'
    # specifies the timeout(in seconds) during which no response from a target means a failed health check
    alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '5'
    # specifies the port used when performing health check on targets
    alb.ingress.kubernetes.io/healthcheck-port: '{{ listen_port }}'
    # redirect traffic from http to https. the ssl-redirect action must be be first rule(which will be evaluated first by ALB)
    alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
    # for creating dns record-set
    external-dns.alpha.kubernetes.io/hostname: {{ domain_name }}
    # specifies the securityGroups you want to attach to LoadBalancer
    alb.ingress.kubernetes.io/security-groups: {{ alb_security_groups }}

  labels:
    app: {{ app_name }}
spec:
  rules:
    - http:
        paths:
          - path: /*
            backend:
              serviceName: ssl-redirect
              servicePort: use-annotation
          - path: /*
            backend:
              serviceName: "{{ app_name }}"
              servicePort: {{ listen_port }}

{% endif %}

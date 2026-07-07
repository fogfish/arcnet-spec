---
"@id": okonkwo-2026-mtls
"@type": source
title: "Deploying mTLS in Service Meshes"
authors: [Chidi Okonkwo]
published: 2026-06-15
url: https://example.org/mtls-service-mesh
created: 2026-06-13
tags: [service-mesh, mtls, operations]
---
# Deploying mTLS in Service Meshes

An operations study of running mutual TLS at scale inside a service mesh, where sidecar
proxies issue and rotate short-lived certificates from a mesh-internal authority, and a
demonstration that automated rotation removes the downtime that historically plagued it.

## Mentions
- mentions:: [[Mutual TLS]]
- mentions:: [[Service Mesh]]
- mentions:: [[Sidecar Proxy]]
- mentions:: [[Certificate Authority]]
- mentions:: [[Identity Provider]]

## Proposes
- proposes:: [[Sidecar-Managed Rotation Is Viable]]

## Raises
- raises:: [[Certificate Rotation Downtime]]

## Cites
- cites:: [[SPIFFE]]

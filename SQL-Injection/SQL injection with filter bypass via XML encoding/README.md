### Lab: SQL injection with filter bypass via XML encoding

## Vulnerability:

*With hackvertor*

*Insert payload into storeId*

**StoreId: entry point, interact directly with database**

*Howerver, WAF will block if we insert UNION. Therefore, we must encoding into hex_intites*

## After doing that, we find account admin and password on request board


WITH parsed_data AS (
    SELECT
        entity_reference_array ->> 'entityId' AS enterpriseid,
        jsonb_array_elements(entity_child_data -> 'entityChildReferenceData') AS childRef,
        entityAttributes->>'specName' AS specName,
        entityAttributes->>'specValue' AS specValue
    FROM
        svcinv.t_service_inventory,
        jsonb_array_elements(entity_reference -> 'entityReferance') entity_reference_array,
        jsonb_array_elements(entity_attributes->'entityAttributes') entityAttributes
    WHERE
        entity_type = 'SBC'
        AND (
            (entity_reference_array ->> 'entityType' = 'ENTERPRISE' AND entity_reference_array ->> 'entityId' = '400304906')
            OR
            (entity_reference_array ->> 'entityType' = 'LOCATION' AND entity_reference_array ->> 'entityId' = '52057')
        )
),
filtered_data AS (
    SELECT
        enterpriseid,
        childRef ->> 'childEntityType' AS childEntityType,
        jsonb_array_elements(cast(childRef -> 'childEntityData' AS jsonb) -> 'childEntityAttributes') AS childEntityAttributes
    FROM
        parsed_data
    WHERE
        childRef ->> 'childEntityType' IN ('DEVICE', 'NODE_INFRA_PROVISIONING', 'SIGNALING_INTERFACE', 'SBC_INFRA_PROVISIONING', 'GATEWAY_SBC')
),
device_filtered AS (
    SELECT
        fd.enterpriseid,
        ca ->> 'specName' AS specName,
        ca ->> 'specValue' AS specValue,
        fd.childEntityAttributes
    FROM
        filtered_data fd,
        LATERAL jsonb_array_elements(fd.childEntityAttributes -> 'attributes') ca
    WHERE
        ca ->> 'specName' = 'DEVICE_ID'
        AND ca ->> 'specValue' = '44'
)
SELECT DISTINCT
    df.enterpriseid,
    ca1 ->> 'specValue' AS DeviceID,
    ca2 ->> 'specValue' AS LocationID,
    ca3 ->> 'specValue' AS RealDeviceType,
    ca4 ->> 'specValue' AS CPECharacteristic,
    ca5 ->> 'specValue' AS CustomerSignalingSubnet,
    ca6 ->> 'specValue' AS CustomerSignalingVLAN,
    ca7 ->> 'specValue' AS CustomerMediaVLAN,
    ca8 ->> 'specValue' AS IP_Version,
    ca9 ->> 'specValue' AS DnsPriority,
    ca10 ->> 'specValue' AS CarrierSignalingFqdn,
    ca101 ->> 'specValue' AS CustomerSignalingBaseIP,
    ca102 ->> 'specValue' AS SbcIndex
FROM
    device_filtered df,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca1,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca2,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca3,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca4,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca5,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca6,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca7,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca8,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca9,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca10,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca101,
    LATERAL jsonb_array_elements(df.childEntityAttributes -> 'attributes') ca102
WHERE
    ca1 ->> 'specName' = 'DEVICE_ID'
    AND ca2 ->> 'specName' = 'LOCATION_ID'
    AND ca3 ->> 'specName' = 'DEVICE_TYPE'
    AND ca4 ->> 'specName' = 'CPE_CHARACTERISTIC'
    AND ca5 ->> 'specName' = 'CUSTOMER_SIGNALING_SUBNET'
    AND ca6 ->> 'specName' = 'CUSTOMER_SIGNALING_VLAN'
    AND ca7 ->> 'specName' = 'CUSTOMER_MEDIA_VLAN'
    AND ca8 ->> 'specName' = 'IP_VERSION'
    AND ca9 ->> 'specName' = 'DNS_PRIORITY'
    AND ca10 ->> 'specName' = 'SIGNALING_FQDN'
    AND ca101 ->> 'specName' = 'CUSTOMER_SIGNALING_BASE_IP'
    AND ca102 ->> 'specName' = 'SBC_INDEX'
    AND (
        (df.specName = 'ROUTER_TYPE' AND df.specValue = 'Type1')
        OR
        (df.specName = 'SWITCH_TYPE' AND df.specValue = 'Type2')
        OR
        (df.specName = 'CARRIER' AND df.specValue = 'Carrier1')
        OR
        (df.specName = 'SIGNALING_GROUP' AND df.specValue = 'Group1')
    );

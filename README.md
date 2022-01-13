-- sp_spaceused 'T_Build'
         SELECT t.NAME AS TableName
              , s.Name AS SchemaName
              , p.rows
              --, SUM(a.total_pages) * 8 AS TotalSpaceKB
              , CAST(ROUND(((SUM(a.total_pages) * 8) / 1024.00 / 1024.00), 2) AS NUMERIC(36, 2)) AS TotalSpaceGB
              --, SUM(a.used_pages) * 8 AS UsedSpaceKB 
              --, CAST(ROUND(((SUM(a.used_pages) * 8) / 1024.00), 2) AS NUMERIC(36, 2)) AS UsedSpaceMB 
              , CAST(ROUND(((SUM(a.used_pages) * 8) / 1024.00 / 1024.00), 2) AS NUMERIC(36, 2)) AS UsedSpaceGB 
              , case when p.rows = 0 then 0 else CAST(ROUND((SUM(a.used_pages) * 8 * 1024 / p.rows), 2) AS NUMERIC(36, 2)) end AS EachRowSizeInBytes
              --, (SUM(a.total_pages) - SUM(a.used_pages)) * 8 AS UnusedSpaceKB
              , CAST(ROUND(((SUM(a.total_pages) - SUM(a.used_pages)) * 8) / 1024.00 / 1024.00, 2) AS NUMERIC(36, 2)) AS UnusedSpaceGB
           FROM sys.tables t
     INNER JOIN sys.indexes i ON t.OBJECT_ID = i.object_id
     INNER JOIN sys.partitions p ON i.object_id = p.OBJECT_ID AND i.index_id = p.index_id
     INNER JOIN sys.allocation_units a ON p.partition_id = a.container_id
LEFT OUTER JOIN sys.schemas s ON t.schema_id = s.schema_id
          WHERE --t.NAME NOT LIKE 'BTC_%' AND 
                t.is_ms_shipped = 0
            AND i.OBJECT_ID > 255 
       GROUP BY t.Name, s.Name, p.Rows
	   order by p.Rows desc

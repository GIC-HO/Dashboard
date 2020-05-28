# CHECKING GICORE UPLOADED ENTRIES

## CHECK GICORE SIDE

Code sau kiểm tra các bút toán có status khác 7 liệu có đuợc upload sang SUNGL hay không. Code trả ra bảng trống nên status 7 trong bảng này đã upload đầy đu sang SUNGL.

```TSQL
WITH
    SUNGL_FILTER AS (
    SELECT --
        DISTINCT
        __.TREFERENCE AS T_REF
    FROM
        dbo.A3_GIC_A_SALFLDG AS __
    WHERE
        1                 = 1
        AND __.JRNAL_TYPE = 'ZZ'
),
    VCHR AS (
    SELECT --
        CONVERT( NUMERIC (19, 0), VHL.GL_FEE_ID )                    AS GL_FEE_ID,
        CONVERT( NVARCHAR, VHL.GL_FEE_ID ) + '_' + --
        CONVERT( NVARCHAR, __.GL_HEAD_ID ) + '_' + --
        CONVERT( NVARCHAR, __.GL_DETAIL_ID )COLLATE DATABASE_DEFAULT AS T_UID,
        CONVERT( NVARCHAR,
                 FORMAT( VHL.TRANS_DATE, 'yyyyMMdd' ) + '_' + --
                 LEFT(VHL.POST_DATE, 12))COLLATE DATABASE_DEFAULT    AS T_REF,
        __.IS_DEBT COLLATE DATABASE_DEFAULT                          AS IS_DEBT,
        CASE
        WHEN __.SEG_1 = '999999'
             THEN '1361108800'
        ELSE __.SEG_1
        END COLLATE DATABASE_DEFAULT                                 AS T_ACCT_CODE
    FROM
        gic_vn_prod_rep.ebao_x.T_GL_VOUCHER_DETAIL_LOG AS __
   INNER JOIN
        gic_vn_prod_rep.ebao_x.T_GL_VOUCHER_HEAD_LOG   AS VHL ON 1                   = 1
                                                                 AND  VHL.GL_HEAD_ID = __.GL_HEAD_ID
    WHERE
        1 = 1
),
    TGLF AS (
    SELECT --
        __.*
    FROM
        gic_vn_prod_rep.ebao_x.T_GL_FEE AS __
    WHERE
        1               = 1
        AND __.STATUS  <> 7
        --AND __.FEE_TYPE = 100500
)
SELECT --
    __.GL_FEE_ID,
    COUNT( SFT.T_REF ) AS T_REF_NO
FROM
    TGLF         AS __
LEFT JOIN
    VCHR         AS GVC ON 1                  = 1
                           AND  GVC.GL_FEE_ID = __.GL_FEE_ID
LEFT JOIN
    SUNGL_FILTER AS SFT ON 1                  = 1
                           AND  SFT.T_REF     = GVC.T_REF
WHERE
    1 = 1
GROUP BY
    __.GL_FEE_ID
HAVING
    COUNT( SFT.T_REF ) <> 0 ;
```

## CHECK SUNGL SIDE

Kiểm tra các bút toán ZZ trên SUNGL có được hoàn toàn upload từ GICORE hay không hoặc có bút toán được sửa bằng tay

```TSQL
DROP TABLE IF EXISTS #TMP ;

SELECT --
    DISTINCT
    CONVERT( NVARCHAR,
             FORMAT( VHL.TRANS_DATE, 'yyyyMMdd' ) + '_' + --
             LEFT(VHL.POST_DATE, 12))COLLATE DATABASE_DEFAULT AS T_REF
INTO
    #TMP
FROM
    gic_vn_prod_rep.ebao_x.T_GL_VOUCHER_DETAIL_LOG AS __
INNER JOIN
    gic_vn_prod_rep.ebao_x.T_GL_VOUCHER_HEAD_LOG   AS VHL ON 1                   = 1
                                                             AND  VHL.GL_HEAD_ID = __.GL_HEAD_ID
WHERE
    1 = 1 ;


WITH
    SUNGL AS (
    SELECT --
        DISTINCT
        __.TREFERENCE AS T_REF
    FROM
        dbo.A3_GIC_A_SALFLDG AS __
    WHERE
        1                 = 1
        AND __.JRNAL_TYPE = 'ZZ'
)
SELECT --
    __.T_REF
FROM
    SUNGL AS __
WHERE
    1 = 1

EXCEPT

SELECT --
    __.T_REF
FROM
    #TMP AS __
WHERE
    1 = 1 ;
```

# tetstet

request url: 
http://agw.sk-iptv.com:8080/metv/v1/purchase/general?IF=IF-ME-031&ver=1.0&response_format=json&stb_id=%7B5E234CCB-7BDD-11E7-AEAA-05066B5ECC13%7D&hash_id=9dcaf463c0d4482f086c1e3dd2f7c653134365d9a273fec29c6769495cedc58f&page_no=2&entry_no=8&resltn_cd=&svc_code=&series_no_resltn_cd=&ui_name=

query string:
?IF=IF-ME-031&ver=1.0&response_format=json&stb_id=%7B5E234CCB-7BDD-11E7-AEAA-05066B5ECC13%7D&hash_id=9dcaf463c0d4482f086c1e3dd2f7c653134365d9a273fec29c6769495cedc58f&page_no=2&entry_no=8&resltn_cd=&svc_code=&series_no_resltn_cd=&ui_name=



첫번째 페이지였을 때, 쿼리

SELECT /*+ MeTV-purchase IF-ME-03x 공통$G2 MeTV WAS$G2 PaaS-구매내역 meta 정보를 CDC DB에서 조회 MONITOR */ 
		    X.PURCHASE_IDX, 
		    X.CID AS ID_CONTENTS, 
		    X.CD_PROD_TYPE, 
		    X.ID_PRODUCT, 
		    X.AMT_PRICE, 
		    X.PURCHASE_PRICE, 
		    X.PURCHASE_CONFIRM_DATE, 
		    X.PURCHASE_END_DATE, 
		    X.AMT_TPOINT_SUP, 
		    X.AMT_CMONEY, 
		    X.AMT_DC_COUPON, 
		    X.AMT_OKCASH, 
		    X.OCB_AMT_VAT, 
		    X.EPSD_NM AS title, -- 에피소드명
		    X.AT_CONTENTS, -- 메타유형코드(009:성인[에로스])
		    X.GRADE_INTERNAL, -- 시청등급코드
		    X.CD_PROD_DEVICE, 
		    DECODE (x.SRIS_TYP_CD, '#', '#', '#') AS YN_SERIES, -- 시리즈 유형 코드(01:시즌, 02:타이틀)
		    X.DD_TELEVISE, -- 방송일자
		    X.ASIS_SRIS_ID AS id_series, -- 시리즈ID
		    X.EXPS_TSEQ_NM AS NO_SERIES, 
		    X.RSLU_TYP_CD AS FG_QUALITY, 
		    DECODE (
		        X.POSSN_YN, 
		        '#', 
		        '#', 
		        DECODE (X.D365_WAT_YN, '#', '#', '#')
		    ) AS unlimitedtype, 
		    X.YN_BROAD_CANCEL, -- 결방여부
		    X.TV_POINT, 
		    DECODE (
		        X.PRD_TYP_CD, '#', '#', '#', '#', 
		        '#'
		    ) AS pdetail_prod_Type, -- 상품유형코드(40:에피소드패키지, 41:시즌패키지)
		    X.PRD_NM AS ppp_title, -- 상품명
		    X.fg_approval AS method_pay_cd, 
		    X.material_cd, -- 배포상태코드
		    DECODE (
		        X.UI4_EPSD_MENU_CD, NULL, X.UI4_SRIS_MENU_CD, 
		        X.UI4_EPSD_MENU_CD
		    ) AS CD_MENU, -- UI4.0기준 메뉴ID
		    X.EPSD_IMG_FILE_PH_NM AS POSTER_URI, -- 에피소드(회차) 포스터 파일 명
		    X.EPSD_IMG_FILE_PW_NM AS SERIES_NO_IMG -- 시리즈 포스터 파일 명
		FROM 
		    (
		        SELECT 
		            pur.*, 
		            C.EPSD_ID, -- 에피소드ID
		            (
		                SELECT 
		                    HEAD_NM 
		                FROM 
		                    BTVCMS.CT_EPSD_POC_DTS 
		                WHERE 
		                    EPSD_ID = C.EPSD_ID 
		                    AND POC_TYP_CD = '#' 
		                    AND HEAD_USE_YN = '#'
		            ) || C.EPSD_NM AS EPSD_NM, 
		            D.WAT_LVL_CD AS GRADE_INTERNAL, 
		            DECODE (
		                D.META_TYP_CD, '#', '#', D.META_TYP_CD
		            ) AS AT_CONTENTS, 
		            J.NSCRN_YN, 
		            DECODE(
		                (
		                    SELECT 
		                        COUNT(#) 
		                    FROM 
		                        BTVCMS.CT_EPSD_POC_DTS 
		                    WHERE 
		                        EPSD_ID = C.EPSD_ID 
		                        AND POC_TYP_CD IN ('#', '#') 
		                        AND NSCRN_YN = '#'
		                ), 
		                #, 
		                '#', 
		                null
		            ) AS CD_PROD_DEVICE, 
		            G.ASIS_SRIS_ID, -- 시리즈ID
		            F.SRIS_TYP_CD, -- 시리즈 유형 코드(01:시즌, 02:타이틀)
		            TO_DATE (D.BRCAST_DY, '#') AS DD_TELEVISE, 
		            E.EXPS_TSEQ_NM, -- 노출회차
		            E.SORT_SEQ, -- 회차정렬순번
		            G.RSLU_TYP_CD, -- 인코딩해상도유형코드
		            B.DIST_STS_CD AS material_cd, 
		            CASE WHEN C.PCIM_ADDN_TYP_CD IS NULL THEN NULL WHEN TO_NUMBER (C.PCIM_ADDN_TYP_CD) BETWEEN # 
		            AND # THEN '#' WHEN TO_NUMBER (C.PCIM_ADDN_TYP_CD) BETWEEN # 
		            AND # THEN '#' ELSE C.PCIM_ADDN_TYP_CD END AS YN_BROAD_CANCEL, 
		            I.POSSN_YN, -- 소장여부
		            I.D365_WAT_YN, -- 365시청여부
		            I.PRD_NM, -- 상품명
		            I.PRD_TYP_CD, -- 상품유형코드(40:에피소드패키지, 41:시즌패키지)
		            (
		                SELECT /*+ LEADING(MER) USE_NL(MM)*/ 
		                    MAX (MM.ASIS_MENU_CD) 
		                FROM 
		                    BTVCMS.MN_MENU_MST MM, 
		                    BTVCMS.MN_MENU_BAS_CNTS_REL MER 
		                WHERE 
		                    MM.MENU_BAS_ID = MER.MENU_BAS_ID 
		                    AND MM.UI_VER_CD = '#' 
		                    AND MM.DEL_YN = '#' 
		                    AND MER.EPSD_ID = C.EPSD_ID
		            ) AS UI4_EPSD_MENU_CD, -- UI4.0기준 에피소드 매핑 메뉴ID
		            (
		                SELECT /*+ LEADING(MER) USE_NL(MM)*/ 
		                    MAX (MM.ASIS_MENU_CD) 
		                FROM 
		                    BTVCMS.MN_MENU_MST MM, 
		                    BTVCMS.MN_MENU_BAS_CNTS_REL MER 
		                WHERE 
		                    MM.MENU_BAS_ID = MER.MENU_BAS_ID 
		                    AND MM.UI_VER_CD = '#' 
		                    AND MM.DEL_YN = '#' 
		                    AND MER.SRIS_ID = F.SRIS_ID
		            ) AS UI4_SRIS_MENU_CD, -- UI4.0기준 시리즈 매핑 메뉴ID
		            (
		                SELECT 
		                    MI.IMG_PATH || MI.IMG_STM_FILE_NM 
		                FROM 
		                    BTVCMS.CT_EPSD_POC_IMG_REL EIR, 
		                    BTVCMS.MT_META_IMG_DTS MI 
		                WHERE 
		                    EIR.META_ID = MI.META_ID 
		                    AND EIR.IMG_ID = MI.IMG_ID 
		                    AND EIR.POC_TYP_CD = '#' 
		                    AND EIR.IMG_TYP_CD = '#' 
		                    AND EIR.EPSD_ID = C.EPSD_ID
		            ) AS EPSD_IMG_FILE_PH_NM, -- 에피소드(회차) 포스터 파일 명
		            (
		                SELECT 
		                    MI.IMG_PATH || MI.IMG_STM_FILE_NM 
		                FROM 
		                    BTVCMS.CT_EPSD_POC_IMG_REL EIR, 
		                    BTVCMS.MT_META_IMG_DTS MI 
		                WHERE 
		                    EIR.META_ID = MI.META_ID 
		                    AND EIR.IMG_ID = MI.IMG_ID 
		                    AND EIR.POC_TYP_CD = '#' 
		                    AND EIR.IMG_TYP_CD = '#' 
		                    AND EIR.EPSD_ID = C.EPSD_ID
		            ) AS EPSD_IMG_FILE_PW_NM -- 시리즈 포스터 파일 명
		        FROM 
		            (
		                SELECT 
		                    PA.PURCHASE_IDX, 
		                    PA.CID, 
		                    PA.CD_PROD_TYPE, 
		                    TO_CHAR (PA.ID_PRODUCT) AS id_product, 
		                    PA.AMT_PRICE, 
		                    PA.PURCHASE_PRICE, 
		                    PA.PURCHASE_CONFIRM_DATE, 
		                    PA.PURCHASE_END_DATE, 
		                    PB.AMT_TPOINT_SUP, 
		                    PB.AMT_CMONEY, 
		                    PB.AMT_DC_COUPON, 
		                    PB.AMT_OKCASH, 
		                    PB.OCB_AMT_VAT, 
		                    NVL (
		                        (
		                            SELECT 
		                                TVP.AMT_APPROVAL 
		                            FROM 
		                                HANARO_SMS.IESV_PURCHASE_LIST_PM TVP 
		                            WHERE 
		                                TVP.PURCHASE_IDX = PA.PURCHASE_IDX 
		                                AND TVP.FG_APPROVAL = ?
		                        ), 
		                        #
		                    ) AS TV_POINT,
		                    PB.fg_approval 
		                FROM 
		                    HANARO_SMS.IESV_PURCHASE_LIST PA, 
		                    HANARO_SMS.IESV_PURCHASE_LIST_PM PB 
		                WHERE 
		                    PA.PURCHASE_IDX = PB.PURCHASE_IDX(+) 
		                    AND PA.CD_PROD_TYPE IN ('#', '#', '#') 
		                    AND PA.PURCHASE_IDX IN  (  ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? ) 
		            ) pur,
		            BTVCMS.CT_EPSD_MST C, 
		            BTVCMS.MT_META_MST D, 
		            BTVCMS.CT_SRIS_EPSD_REL E, 
		            BTVCMS.CT_SRIS_MST F, 
		            BTVCMS.CT_EPSD_RSLU_MST G, 
		            BTVCMS.PD_PRD_PRC_DTS H, 
		            BTVCMS.PD_PRD_MST I, 
		            BTVCMS.CT_EPSD_POC_DTS J, 
		            BTVCMS.CT_EPSD_STS_MRX_DTS B 
		        WHERE 
		            pur.CID = G.EPSD_RSLU_ID 
		            AND G.EPSD_ID = C.EPSD_ID 
		            AND C.META_ID = D.META_ID 
		            AND C.EPSD_ID = E.EPSD_ID 
		            AND E.SRIS_ID = F.SRIS_ID 
		            AND pur.ID_PRODUCT = H.PRD_PRC_ID 
		            AND H.PRD_ID = I.PRD_ID 
		            AND C.EPSD_ID = B.EPSD_ID(+) 
		            AND B.POC_TYP_CD(+) = '#' 
		            AND C.DEL_YN = '#' 
		            AND D.DEL_YN = '#' 
		            AND F.DEL_YN = '#' 
		            AND G.DEL_YN = '#' 
		            AND I.DEL_YN = '#' 
		            AND C.EPSD_ID = J.EPSD_ID 
		            AND J.POC_TYP_CD = '#' 
		            AND F.SRIS_TYP_CD IN ('#', '#')
		    ) X 
		ORDER BY 
		    X.PURCHASE_CONFIRM_DATE DESC


두번째 페이지였을 때, 쿼리

SELECT /*+ MeTV-purchase IF-ME-03x 공통$G2 MeTV WAS$G2 PaaS-구매내역 meta 정보를 CDC DB에서 조회 MONITOR */ 
		    X.PURCHASE_IDX, 
		    X.CID AS ID_CONTENTS, 
		    X.CD_PROD_TYPE, 
		    X.ID_PRODUCT, 
		    X.AMT_PRICE, 
		    X.PURCHASE_PRICE, 
		    X.PURCHASE_CONFIRM_DATE, 
		    X.PURCHASE_END_DATE, 
		    X.AMT_TPOINT_SUP, 
		    X.AMT_CMONEY, 
		    X.AMT_DC_COUPON, 
		    X.AMT_OKCASH, 
		    X.OCB_AMT_VAT, 
		    X.EPSD_NM AS title, -- 에피소드명
		    X.AT_CONTENTS, -- 메타유형코드(009:성인[에로스])
		    X.GRADE_INTERNAL, -- 시청등급코드
		    X.CD_PROD_DEVICE, 
		    DECODE (x.SRIS_TYP_CD, '#', '#', '#') AS YN_SERIES, -- 시리즈 유형 코드(01:시즌, 02:타이틀)
		    X.DD_TELEVISE, -- 방송일자
		    X.ASIS_SRIS_ID AS id_series, -- 시리즈ID
		    X.EXPS_TSEQ_NM AS NO_SERIES, 
		    X.RSLU_TYP_CD AS FG_QUALITY, 
		    DECODE (
		        X.POSSN_YN, 
		        '#', 
		        '#', 
		        DECODE (X.D365_WAT_YN, '#', '#', '#')
		    ) AS unlimitedtype, 
		    X.YN_BROAD_CANCEL, -- 결방여부
		    X.TV_POINT, 
		    DECODE (
		        X.PRD_TYP_CD, '#', '#', '#', '#', 
		        '#'
		    ) AS pdetail_prod_Type, -- 상품유형코드(40:에피소드패키지, 41:시즌패키지)
		    X.PRD_NM AS ppp_title, -- 상품명
		    X.fg_approval AS method_pay_cd, 
		    X.material_cd, -- 배포상태코드
		    DECODE (
		        X.UI4_EPSD_MENU_CD, NULL, X.UI4_SRIS_MENU_CD, 
		        X.UI4_EPSD_MENU_CD
		    ) AS CD_MENU, -- UI4.0기준 메뉴ID
		    X.EPSD_IMG_FILE_PH_NM AS POSTER_URI, -- 에피소드(회차) 포스터 파일 명
		    X.EPSD_IMG_FILE_PW_NM AS SERIES_NO_IMG -- 시리즈 포스터 파일 명
		FROM 
		    (
		        SELECT 
		            pur.*, 
		            C.EPSD_ID, -- 에피소드ID
		            (
		                SELECT 
		                    HEAD_NM 
		                FROM 
		                    BTVCMS.CT_EPSD_POC_DTS 
		                WHERE 
		                    EPSD_ID = C.EPSD_ID 
		                    AND POC_TYP_CD = '#' 
		                    AND HEAD_USE_YN = '#'
		            ) || C.EPSD_NM AS EPSD_NM, 
		            D.WAT_LVL_CD AS GRADE_INTERNAL, 
		            DECODE (
		                D.META_TYP_CD, '#', '#', D.META_TYP_CD
		            ) AS AT_CONTENTS, 
		            J.NSCRN_YN, 
		            DECODE(
		                (
		                    SELECT 
		                        COUNT(#) 
		                    FROM 
		                        BTVCMS.CT_EPSD_POC_DTS 
		                    WHERE 
		                        EPSD_ID = C.EPSD_ID 
		                        AND POC_TYP_CD IN ('#', '#') 
		                        AND NSCRN_YN = '#'
		                ), 
		                #, 
		                '#', 
		                null
		            ) AS CD_PROD_DEVICE, 
		            G.ASIS_SRIS_ID, -- 시리즈ID
		            F.SRIS_TYP_CD, -- 시리즈 유형 코드(01:시즌, 02:타이틀)
		            TO_DATE (D.BRCAST_DY, '#') AS DD_TELEVISE, 
		            E.EXPS_TSEQ_NM, -- 노출회차
		            E.SORT_SEQ, -- 회차정렬순번
		            G.RSLU_TYP_CD, -- 인코딩해상도유형코드
		            B.DIST_STS_CD AS material_cd, 
		            CASE WHEN C.PCIM_ADDN_TYP_CD IS NULL THEN NULL WHEN TO_NUMBER (C.PCIM_ADDN_TYP_CD) BETWEEN # 
		            AND # THEN '#' WHEN TO_NUMBER (C.PCIM_ADDN_TYP_CD) BETWEEN # 
		            AND # THEN '#' ELSE C.PCIM_ADDN_TYP_CD END AS YN_BROAD_CANCEL, 
		            I.POSSN_YN, -- 소장여부
		            I.D365_WAT_YN, -- 365시청여부
		            I.PRD_NM, -- 상품명
		            I.PRD_TYP_CD, -- 상품유형코드(40:에피소드패키지, 41:시즌패키지)
		            (
		                SELECT /*+ LEADING(MER) USE_NL(MM)*/ 
		                    MAX (MM.ASIS_MENU_CD) 
		                FROM 
		                    BTVCMS.MN_MENU_MST MM, 
		                    BTVCMS.MN_MENU_BAS_CNTS_REL MER 
		                WHERE 
		                    MM.MENU_BAS_ID = MER.MENU_BAS_ID 
		                    AND MM.UI_VER_CD = '#' 
		                    AND MM.DEL_YN = '#' 
		                    AND MER.EPSD_ID = C.EPSD_ID
		            ) AS UI4_EPSD_MENU_CD, -- UI4.0기준 에피소드 매핑 메뉴ID
		            (
		                SELECT /*+ LEADING(MER) USE_NL(MM)*/ 
		                    MAX (MM.ASIS_MENU_CD) 
		                FROM 
		                    BTVCMS.MN_MENU_MST MM, 
		                    BTVCMS.MN_MENU_BAS_CNTS_REL MER 
		                WHERE 
		                    MM.MENU_BAS_ID = MER.MENU_BAS_ID 
		                    AND MM.UI_VER_CD = '#' 
		                    AND MM.DEL_YN = '#' 
		                    AND MER.SRIS_ID = F.SRIS_ID
		            ) AS UI4_SRIS_MENU_CD, -- UI4.0기준 시리즈 매핑 메뉴ID
		            (
		                SELECT 
		                    MI.IMG_PATH || MI.IMG_STM_FILE_NM 
		                FROM 
		                    BTVCMS.CT_EPSD_POC_IMG_REL EIR, 
		                    BTVCMS.MT_META_IMG_DTS MI 
		                WHERE 
		                    EIR.META_ID = MI.META_ID 
		                    AND EIR.IMG_ID = MI.IMG_ID 
		                    AND EIR.POC_TYP_CD = '#' 
		                    AND EIR.IMG_TYP_CD = '#' 
		                    AND EIR.EPSD_ID = C.EPSD_ID
		            ) AS EPSD_IMG_FILE_PH_NM, -- 에피소드(회차) 포스터 파일 명
		            (
		                SELECT 
		                    MI.IMG_PATH || MI.IMG_STM_FILE_NM 
		                FROM 
		                    BTVCMS.CT_EPSD_POC_IMG_REL EIR, 
		                    BTVCMS.MT_META_IMG_DTS MI 
		                WHERE 
		                    EIR.META_ID = MI.META_ID 
		                    AND EIR.IMG_ID = MI.IMG_ID 
		                    AND EIR.POC_TYP_CD = '#' 
		                    AND EIR.IMG_TYP_CD = '#' 
		                    AND EIR.EPSD_ID = C.EPSD_ID
		            ) AS EPSD_IMG_FILE_PW_NM -- 시리즈 포스터 파일 명
		        FROM 
		            (
		                SELECT 
		                    PA.PURCHASE_IDX, 
		                    PA.CID, 
		                    PA.CD_PROD_TYPE, 
		                    TO_CHAR (PA.ID_PRODUCT) AS id_product, 
		                    PA.AMT_PRICE, 
		                    PA.PURCHASE_PRICE, 
		                    PA.PURCHASE_CONFIRM_DATE, 
		                    PA.PURCHASE_END_DATE, 
		                    PB.AMT_TPOINT_SUP, 
		                    PB.AMT_CMONEY, 
		                    PB.AMT_DC_COUPON, 
		                    PB.AMT_OKCASH, 
		                    PB.OCB_AMT_VAT, 
		                    NVL (
		                        (
		                            SELECT 
		                                TVP.AMT_APPROVAL 
		                            FROM 
		                                HANARO_SMS.IESV_PURCHASE_LIST_PM TVP 
		                            WHERE 
		                                TVP.PURCHASE_IDX = PA.PURCHASE_IDX 
		                                AND TVP.FG_APPROVAL = ?
		                        ), 
		                        #
		                    ) AS TV_POINT,
		                    PB.fg_approval 
		                FROM 
		                    HANARO_SMS.IESV_PURCHASE_LIST PA, 
		                    HANARO_SMS.IESV_PURCHASE_LIST_PM PB 
		                WHERE 
		                    PA.PURCHASE_IDX = PB.PURCHASE_IDX(+) 
		                    AND PA.CD_PROD_TYPE IN ('#', '#', '#') 
		                    AND PA.PURCHASE_IDX IN  (  ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? , ? ) 
		            ) pur,
		            BTVCMS.CT_EPSD_MST C, 
		            BTVCMS.MT_META_MST D, 
		            BTVCMS.CT_SRIS_EPSD_REL E, 
		            BTVCMS.CT_SRIS_MST F, 
		            BTVCMS.CT_EPSD_RSLU_MST G, 
		            BTVCMS.PD_PRD_PRC_DTS H, 
		            BTVCMS.PD_PRD_MST I, 
		            BTVCMS.CT_EPSD_POC_DTS J, 
		            BTVCMS.CT_EPSD_STS_MRX_DTS B 
		        WHERE 
		            pur.CID = G.EPSD_RSLU_ID 
		            AND G.EPSD_ID = C.EPSD_ID 
		            AND C.META_ID = D.META_ID 
		            AND C.EPSD_ID = E.EPSD_ID 
		            AND E.SRIS_ID = F.SRIS_ID 
		            AND pur.ID_PRODUCT = H.PRD_PRC_ID 
		            AND H.PRD_ID = I.PRD_ID 
		            AND C.EPSD_ID = B.EPSD_ID(+) 
		            AND B.POC_TYP_CD(+) = '#' 
		            AND C.DEL_YN = '#' 
		            AND D.DEL_YN = '#' 
		            AND F.DEL_YN = '#' 
		            AND G.DEL_YN = '#' 
		            AND I.DEL_YN = '#' 
		            AND C.EPSD_ID = J.EPSD_ID 
		            AND J.POC_TYP_CD = '#' 
		            AND F.SRIS_TYP_CD IN ('#', '#')
		    ) X 
		ORDER BY 
		    X.PURCHASE_CONFIRM_DATE DESC

  
        
        
        
        

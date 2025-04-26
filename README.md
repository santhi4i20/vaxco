1
CREATE OR REPLACE FUNCTION fnc_valor_total_pedidos_por_estado(p_uf IN VARCHAR2)

RETURN NUMBER

IS
    v_valor_total    NUMBER;

BEGIN



    SELECT SUM(HP.VAL_TOTAL_PEDIDO)
    INTO v_valor_total
    FROM HISTORICO_PEDIDO HP
    JOIN ENDERECO_CLIENTE EC ON EC.COD_CLIENTE = HP.COD_CLIENTE
    JOIN CIDADE C ON C.COD_CIDADE = EC.COD_CIDADE
    JOIN ESTADO E ON E.COD_ESTADO = C.COD_ESTADO
    WHERE E.NOM_ESTADO = p_uf
    AND HP.DAT_ENTREGA IS NOT NULL;



    IF v_valor_total IS NULL THEN
        RETURN 0;
    ELSE
        RETURN v_valor_total;
    END IF;



EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 0;

    WHEN OTHERS THEN
        RETURN 0;

END;


2
CREATE OR REPLACE FUNCTION fnc_qtd_itens_em_pedidos_por_produto(p_cod_produto IN NUMBER)

RETURN NUMBER

IS
    v_total_unidades   NUMBER;

BEGIN



    SELECT SUM(IP.QTD_ITEM)
    INTO v_total_unidades
    FROM ITEM_PEDIDO IP
    JOIN HISTORICO_PEDIDO HP ON HP.COD_PEDIDO = IP.COD_PEDIDO
    WHERE IP.COD_PRODUTO = p_cod_produto;



    IF v_total_unidades IS NULL THEN
        RETURN 0;
    ELSE
        RETURN v_total_unidades;
    END IF;



EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 0;

    WHEN OTHERS THEN
        RETURN 0;

END;


3
CREATE OR REPLACE PROCEDURE prc_relatorio_pedidos_por_cliente

IS

BEGIN



    FOR cliente IN (

        SELECT 
            C.COD_CLIENTE,
            C.NOM_CLIENTE,
            CID.NOM_CIDADE,
            COUNT(HP.COD_PEDIDO) AS QTD_PEDIDOS,
            NVL(SUM(HP.VAL_TOTAL_PEDIDO), 0) AS VALOR_TOTAL,
            MAX(CASE WHEN HP.DAT_CANCELAMENTO IS NOT NULL THEN 1 ELSE 0 END) AS TEM_CANCELADO

        FROM CLIENTE C
        LEFT JOIN ENDERECO_CLIENTE EC ON EC.COD_CLIENTE = C.COD_CLIENTE
        LEFT JOIN CIDADE CID ON CID.COD_CIDADE = EC.COD_CIDADE
        LEFT JOIN HISTORICO_PEDIDO HP ON HP.COD_CLIENTE = C.COD_CLIENTE

        GROUP BY C.COD_CLIENTE, C.NOM_CLIENTE, CID.NOM_CIDADE

    ) LOOP



        DBMS_OUTPUT.PUT_LINE('Cliente: ' || cliente.NOM_CLIENTE);
        DBMS_OUTPUT.PUT_LINE('Cidade: ' || NVL(cliente.NOM_CIDADE, 'Sem Cidade'));
        DBMS_OUTPUT.PUT_LINE('Qtd. Pedidos: ' || cliente.QTD_PEDIDOS);
        DBMS_OUTPUT.PUT_LINE('Valor Total Comprado: R$ ' || cliente.VALOR_TOTAL);



        IF cliente.TEM_CANCELADO = 1 THEN
            DBMS_OUTPUT.PUT_LINE('Possui pedidos cancelados.');
        ELSE
            DBMS_OUTPUT.PUT_LINE('Nenhum pedido cancelado.');
        END IF;



        DBMS_OUTPUT.PUT_LINE('-------------------------------');



    END LOOP;



EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Erro inesperado: ' || SQLERRM);

END;


4
CREATE OR REPLACE PROCEDURE prc_movimentacao_produto_por_vendedor

IS

BEGIN



    FOR vendedor IN (

        SELECT 
            V.COD_VENDEDOR
        FROM CLIENTE_VENDEDOR V

    ) LOOP



        DBMS_OUTPUT.PUT_LINE('Vendedor Código: ' || vendedor.COD_VENDEDOR);



        FOR produto IN (

            SELECT 
                IP.COD_PRODUTO,
                SUM(IP.QTD_ITEM) AS QTD_TOTAL,
                SUM(IP.QTD_ITEM * IP.VAL_UNITARIO_ITEM) AS VALOR_TOTAL

            FROM HISTORICO_PEDIDO HP
            JOIN ITEM_PEDIDO IP ON IP.COD_PEDIDO = HP.COD_PEDIDO

            WHERE HP.COD_VENDEDOR = vendedor.COD_VENDEDOR
            GROUP BY IP.COD_PRODUTO

        ) LOOP



            DBMS_OUTPUT.PUT_LINE('Produto Código: ' || produto.COD_PRODUTO);
            DBMS_OUTPUT.PUT_LINE('Quantidade Vendida: ' || NVL(produto.QTD_TOTAL, 0));
            DBMS_OUTPUT.PUT_LINE('Valor Total Vendido: R$ ' || NVL(produto.VALOR_TOTAL, 0));
            DBMS_OUTPUT.PUT_LINE('-------------------------');



        END LOOP;



        IF SQL%ROWCOUNT = 0 THEN
            DBMS_OUTPUT.PUT_LINE('Sem vendas registradas.');
        END IF;



        DBMS_OUTPUT.PUT_LINE('=================================');



    END LOOP;



EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Erro inesperado: ' || SQLERRM);

END;


5
CREATE OR REPLACE PROCEDURE prc_analise_vendas_por_vendedor(p_cod_vendedor IN NUMBER)

IS

BEGIN



    FOR cliente IN (

        SELECT 
            C.NOM_CLIENTE,
            PR.COD_PRODUTO,
            SUM(IP.QTD_ITEM) AS TOTAL_COMPRADO

        FROM CLIENTE C
        LEFT JOIN HISTORICO_PEDIDO HP ON HP.COD_CLIENTE = C.COD_CLIENTE
        LEFT JOIN ITEM_PEDIDO IP ON IP.COD_PEDIDO = HP.COD_PEDIDO
        LEFT JOIN ESTOQUE_PRODUTO EP ON EP.COD_PRODUTO = IP.COD_PRODUTO

        WHERE HP.COD_VENDEDOR = p_cod_vendedor
        GROUP BY C.NOM_CLIENTE, PR.COD_PRODUTO

    ) LOOP



        DBMS_OUTPUT.PUT_LINE('Cliente: ' || NVL(cliente.NOM_CLIENTE, 'Sem Nome'));
        DBMS_OUTPUT.PUT_LINE('Produto Código: ' || NVL(cliente.COD_PRODUTO, 'Sem Produto'));
        DBMS_OUTPUT.PUT_LINE('Quantidade Comprada: ' || NVL(cliente.TOTAL_COMPRADO, 0));



        IF cliente.TOTAL_COMPRADO > 50 THEN
            DBMS_OUTPUT.PUT_LINE('Perfil: CLIENTE FIEL');

        ELSIF cliente.TOTAL_COMPRADO BETWEEN 11 AND 50 THEN
            DBMS_OUTPUT.PUT_LINE('Perfil: CLIENTE RECORRENTE');

        ELSIF cliente.TOTAL_COMPRADO BETWEEN 1 AND 10 THEN
            DBMS_OUTPUT.PUT_LINE('Perfil: CLIENTE OCASIONAL');

        ELSE
            DBMS_OUTPUT.PUT_LINE('Perfil: NENHUMA COMPRA REGISTRADA');
        END IF;



        DBMS_OUTPUT.PUT_LINE('------------------------------');



    END LOOP;



EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Erro inesperado: ' || SQLERRM);

END;







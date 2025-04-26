1. Função – fnc_valor_total_por_estado
Implemente uma função chamada fnc_valor_total_pedidos_por_estado que receba a sigla de um estado como parâmetro (p_uf IN VARCHAR2). Essa função deve consultar todos os pedidos vinculados a clientes residentes nesse estado e retornar o valor total de todos os pedidos que tenham entrega confirmada (DAT_ENTREGA IS NOT NULL).

Essa função é utilizada para medir o desempenho de vendas por estado, sendo essencial para análise regional. A consulta deverá realizar JOINs entre as tabelas historico_pedido, endereco_cliente, cidade e estado. Deverá aplicar SUM para calcular o valor total e utilizar filtros para entregar apenas pedidos entregues.

Parâmetro: p_uf VARCHAR2
Retorno: NUMBER (soma dos valores de pedidos entregues)

2. Função – fnc_qtd_itens_em_pedidos_por_produto
Crie uma função chamada fnc_qtd_itens_em_pedidos_por_produto que receba o código de um produto (p_cod_produto IN NUMBER) e retorne a quantidade total de unidades vendidas desse produto. A função deve utilizar JOINs entre item_pedido, pedido e produto, e aplicar agregação com SUM na coluna QTD.

Parâmetro: p_cod_produto NUMBER
Retorno: NUMBER (quantidade total vendida)

3. Procedure – prc_relatorio_pedidos_por_cliente
Desenvolva a procedure prc_relatorio_pedidos_por_cliente, que percorre todos os clientes utilizando um FOR LOOP. Para cada cliente, deve exibir via DBMS_OUTPUT: nome do cliente, cidade, quantidade de pedidos realizados, valor total comprado e se houve algum pedido cancelado (com IF/ELSE). Clientes sem pedidos também devem ser incluídos usando LEFT JOIN. Trate exceções como clientes sem endereço ou sem cidade associada.

Parâmetro: Nenhum
Saída: Exibição formatada via DBMS_OUTPUT.PUT_LINE


4. Procedure – prc_movimentacao_produto_por_vendedor
Crie a procedure prc_movimentacao_produto_por_vendedor, que percorre cada vendedor e lista os produtos vendidos, quantidade total e valor total de vendas por produto. Utilize um LOOP para percorrer os produtos por vendedor e IF para mostrar "Sem vendas registradas" quando aplicável. Inclua vendedores sem movimentações usando RIGHT JOIN. Trate exceções relacionadas a dados faltantes.

Esse exercício foca em indicadores de performance individuais e exige o uso de controle de fluxo e agregações condicionais.

Parâmetro: Nenhum
Saída: Exibição via DBMS_OUTPUT


5. Procedure – prc_analise_vendas_por_vendedor
   Implemente a procedure prc_analise_vendas_por_vendedor, que recebe como parâmetro o código de um vendedor (p_cod_vendedor NUMBER) e apresenta um relatório cruzando informações de clientes, produtos e vendas realizadas por esse vendedor. A procedure deve utilizar JOINs entre as tabelas vendedor, pedido, item_pedido, produto e cliente para obter, para cada cliente atendido pelo vendedor informado: o nome do cliente, o nome do produto adquirido e a quantidade total comprada. Os dados devem ser agrupados por cliente e produto, e exibidos por meio de DBMS_OUTPUT.PUT_LINE. A procedure deve percorrer os clientes utilizando um LOOP, aplicar a estrutura IF/ELSE para classificar o perfil de compra de cada cliente com base na quantidade total adquirida: exiba “CLIENTE FIEL” para clientes que compraram mais de 50 unidades, “CLIENTE RECORRENTE” para aqueles que compraram entre 11 e 50 unidades, “CLIENTE OCASIONAL” para até 10 unidades e “NENHUMA COMPRA REGISTRADA” caso o cliente não tenha efetuado nenhuma compra. Utilize LEFT JOIN para garantir que clientes sem pedidos também sejam considerados na análise. Implemente tratamento de exceções utilizando EXCEPTION para lidar com casos de dados nulos, clientes ou produtos com nome ausente e eventuais erros de execução. Essa procedure permite gerar uma análise personalizada de comportamento de compra por vendedor, sendo útil para estratégias de fidelização e gestão de performance comercial.
Parâmetro: p_cod_vendedor NUMBER
Saída: Exibição via DBMS_OUTPUT

CREATE OR REPLACE FUNCTION fnc_valor_total_pedidos_por_estado(p_uf IN VARCHAR2)

RETURN NUMBER

IS
    v_valor_total    NUMBER;

BEGIN



    SELECT SUM(HP.VAL_TOTAL_PEDIDO)
    INTO v_valor_total
    FROM HISTORICO_PEDIDO HP
    JOIN PEDIDO P ON P.COD_PEDIDO = HP.COD_PEDIDO
    JOIN ENDERECO_CLIENTE EC ON EC.COD_CLIENTE = P.COD_CLIENTE
    JOIN CIDADE C ON C.COD_CIDADE = EC.COD_CIDADE
    JOIN ESTADO E ON E.COD_ESTADO = C.COD_ESTADO
    WHERE E.UF = p_uf
    AND P.DAT_ENTREGA IS NOT NULL;



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



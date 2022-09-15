# BRH #

> STATUS DO PROJETO: EM DESENVOLVIMENTO

A intenção do projeto é desenvolver um software para o RH.


As principais entidades de relacionamento são:
* Departamento;
* Colaborador;
* Papel;
* Projeto;
* Dependente.

´´´ Alguns relatórios importantes 
~~~ MySQL
/* RELATORIO DE DEPARTAMENTO */
SELECT NOME,SIGLA FROM DEPARTAMENTO
ORDER BY NOME;

/* EXERCICIO RELATORIO DE DEPENDENTES */
SELECT COLABORADOR.NOME AS 'NOME_COLABORADOR',DEPENDENTE.NOME AS 'NOME_DEPENDENTE', DEPENDENTE.DATA_NASCIMENTO, DEPENDENTE.PARENTESCO FROM DEPENDENTE, COLABORADOR
WHERE DEPENDENTE.COLABORADOR = COLABORADOR.MATRICULA
ORDER BY COLABORADOR.NOME
;

/* OU */

SELECT COLABORADOR.NOME AS 'NOME_COLABORADOR',DEPENDENTE.NOME AS 'NOME_DEPENDENTE', DEPENDENTE.DATA_NASCIMENTO, DEPENDENTE.PARENTESCO FROM DEPENDENTE INNER JOIN COLABORADOR
ON DEPENDENTE.COLABORADOR = COLABORADOR.MATRICULA
ORDER BY COLABORADOR.NOME;
~~~



~~~ Oracle
/* EXERCICIO RELATORIO DE DEPARTAMENTO - ORACLE */
SELECT NOME,SIGLA FROM BRH.DEPARTAMENTO;



/* EXERCICIO RELATORIO DE DEPENDENTES - ORACLE */
SELECT BRH.COLABORADOR.NOME AS NOME_COLABORADOR, BRH.DEPENDENTE.NOME AS NOME_DEPENDENTE,BRH.DEPENDENTE.DATA_NASCIMENTO, BRH.DEPENDENTE.PARENTESCO  FROM BRH.COLABORADOR, BRH.DEPENDENTE WHERE BRH.DEPENDENTE.COLABORADOR = BRH.COLABORADOR.MATRICULA ;


 
/* RELATÓRIO DE CONTATOS */
SELECT BRH.COLABORADOR.NOME AS NOME_COLABORADOR, BRH.COLABORADOR.EMAIL_CORPORATIVO,BRH.TELEFONE_COLABORADOR.NUMERO FROM BRH.COLABORADOR INNER JOIN BRH.TELEFONE_COLABORADOR
ON BRH.COLABORADOR.MATRICULA = BRH.TELEFONE_COLABORADOR.COLABORADOR
WHERE BRH.TELEFONE_COLABORADOR.TIPO = 'M'
ORDER BY BRH.COLABORADOR.NOME   



/* RELATÓRIO ANALITICO DE EQUIPES */
SELECT B.NOME NOME_DEPARTAMENTO,B.CHEFE CHEFE_DEPARTAMENTO, ALOCACAO.NOME,P.NOME NOME_PROJETO,PAP.NOME PAPEL,TC.NUMERO NUMERO, DEP.NOME DEPENDENTE FROM BRH.DEPARTAMENTO B
INNER JOIN BRH.COLABORADOR CHEFE
ON B.CHEFE = CHEFE.MATRICULA
INNER JOIN BRH.COLABORADOR ALOCACAO
ON B.SIGLA = ALOCACAO.DEPARTAMENTO
INNER JOIN BRH.ATRIBUICAO AL
ON AL.COLABORADOR = ALOCACAO.MATRICULA
INNER JOIN BRH.PROJETO P
ON P.ID = AL.PROJETO
INNER JOIN BRH.PAPEL PAP
ON PAP.ID = AL.PAPEL
INNER JOIN BRH.TELEFONE_COLABORADOR TC
ON TC.COLABORADOR = ALOCACAO.MATRICULA 
LEFT JOIN BRH.DEPENDENTE DEP
ON DEP.COLABORADOR = ALOCACAO.MATRICULA
/* PARA CONSULTAR SOMENTE O MOVEL 
WHERE TC.TIPO = 'M' */
/* PARA SABER SOMENTE O COMERCIAL
WHERE TC.TIPO = 'C' */
ORDER BY B.NOME, ALOCACAO.NOME



/* FILTRAR DEPENDENTES */
SELECT  BRH.COLABORADOR.NOME AS NOME_COLABORADOR, 
BRH.DEPENDENTE.NOME AS NOME_DEPENDENTE, 
BRH.DEPENDENTE.DATA_NASCIMENTO, 
BRH.DEPENDENTE.PARENTESCO 
FROM BRH.DEPENDENTE 
LEFT JOIN BRH.COLABORADOR 
ON BRH.DEPENDENTE.COLABORADOR = BRH.COLABORADOR.MATRICULA
WHERE EXTRACT (MONTH FROM BRH.DEPENDENTE.DATA_NASCIMENTO) >= 04 
AND 
EXTRACT (MONTH FROM BRH.DEPENDENTE.DATA_NASCIMENTO) <= 06
OR 
BRH.DEPENDENTE.NOME LIKE '%H%'
ORDER BY NOME_COLABORADOR;



/* LISTAR O COLABORADOR COM O MAIOR SALARIO*/ 
SELECT * FROM BRH.COLABORADOR 
WHERE BRH.COLABORADOR.SALARIO = 
(SELECT MAX(BRH.COLABORADOR.SALARIO) 
FROM BRH.COLABORADOR);



/* RELATÓRIO SENIORIDADE */
SELECT A.MATRICULA, 
A.NOME, 
A.SALARIO, 
(CASE 
WHEN A.SALARIO <=3000 THEN 'JÚNIOR' 
WHEN A.SALARIO >= 3001 AND A.SALARIO <=6000 THEN 'PLENO'
WHEN A.SALARIO >= 6001 AND A.SALARIO <=20000 THEN 'SÊNIOR'
WHEN A.SALARIO >20000 THEN 'CORPO DIRETOR'
END ) AS SENIORIDADE
FROM BRH.COLABORADOR A
ORDER BY SENIORIDADE,A.NOME;



/* LISTAR OS COLABORADORES EM PROJETOS */

SELECT B.NOME AS NOME_DEP, 
E.NOME AS NOME_PROJETO, 
COUNT (*) AS QTD_PESSOAS 
FROM BRH.PROJETO E
INNER JOIN BRH.ATRIBUICAO D
ON E.ID = D.PROJETO
INNER JOIN BRH.COLABORADOR A
ON A.MATRICULA = D.COLABORADOR
INNER JOIN BRH.DEPARTAMENTO B
ON B.SIGLA = A.DEPARTAMENTO
GROUP BY B.NOME,
E.NOME
ORDER BY B.NOME,
E.NOME;



/* RELATÓRIO COLABORADOR COM MAIS DEPENDENTES */
SELECT A.NOME, 
COUNT (*) AS QTD_DEPENDENTES 
FROM BRH.DEPENDENTE B 
INNER JOIN BRH.COLABORADOR A
ON A.MATRICULA = B.COLABORADOR
GROUP BY A.NOME,B.COLABORADOR
HAVING COUNT (*) >= 2
ORDER BY QTD_DEPENDENTES DESC,
A.NOME ASC;



/* LISTAR FAIXA ETARIA DOS DEPENDENTES */
SELECT B.CPF, 
B.NOME AS NOME_DEPENDENTE, 
B.DATA_NASCIMENTO, 
B.PARENTESCO, 
A.MATRICULA, 
TRUNC (to_char(SYSDATE - B.DATA_NASCIMENTO)/365.25) as IDADE,
(CASE 
WHEN (SYSDATE - B.DATA_NASCIMENTO)/365 <18 THEN 'MENOR DE IDADE'
ELSE 'MAIOR DE IDADE'
END ) AS FAIXA_ETARIA
FROM BRH.COLABORADOR A
INNER JOIN BRH.DEPENDENTE B
ON A.MATRICULA = B.COLABORADOR
ORDER BY A.MATRICULA, 
B.NOME;



/* RELATÓRIO PLANO DE SAÚDE */

/* VW_SENI_IDADE */
SELECT B.CPF, 
B.NOME AS NOME_DEPENDENTE,B.PARENTESCO, 
B.DATA_NASCIMENTO, 
TRUNC (TO_CHAR(SYSDATE - B.DATA_NASCIMENTO)/365.25) as IDADE,
(CASE 
WHEN (SYSDATE - B.DATA_NASCIMENTO)/365 <18 THEN 'MENOR DE IDADE'
ELSE 'MAIOR DE IDADE'
END ) AS FAIXA_ETARIA, 
A.MATRICULA,
A.NOME AS NOME_COLABORADOR, 
A.SALARIO, 
(CASE 
WHEN A.SALARIO <=3000 THEN 'JÚNIOR' 
WHEN A.SALARIO >= 3001 AND A.SALARIO <=6000 THEN 'PLENO'
WHEN A.SALARIO >= 6001 AND A.SALARIO <=20000 THEN 'SÊNIOR'
WHEN A.SALARIO >20000 THEN 'CORPO DIRETOR'
END ) AS SENIORIDADE
FROM BRH.COLABORADOR A
LEFT JOIN BRH.DEPENDENTE B
ON A.MATRICULA = B.COLABORADOR
ORDER BY A.MATRICULA, 
B.NOME, 
SENIORIDADE;
/* VW_SENI_IDADE */



/* VW_PAGAR_SENIORIDADE */
SELECT DISTINCT NOME_COLABORADOR, 
MATRICULA, 
(CASE
WHEN SENIORIDADE = 'CORPO DIRETOR' THEN SALARIO*0.05
WHEN SENIORIDADE = 'SÊNIOR' THEN SALARIO*0.03
WHEN SENIORIDADE = 'PLENO' THEN SALARIO*0.02
WHEN SENIORIDADE = 'JÚNIOR' THEN SALARIO*0.01
END ) AS PS
FROM VW_SENI_IDADE;
/* VW_PAGAR_SENIORIDADE */



/* VW_PS_DEPENDENTE */
SELECT V.MATRICULA, V.NOME_COLABORADOR, SUM(
CASE 
WHEN PARENTESCO = 'CÃ´njuge' THEN ('100')
WHEN IDADE <18 THEN ('25')
WHEN IDADE >= 18 THEN ('50')
END ) AS PLANO_SAUDE 
FROM VW_SENI_IDADE V
INNER JOIN VW_PAGAR_SENIORIDADE P
ON P.MATRICULA = V.MATRICULA 
GROUP BY V.MATRICULA, 
V.NOME_COLABORADOR 
ORDER BY V.NOME_COLABORADOR;
/* VW_PS_DEPENDENTE */



/* SELECT PARA TRAZER OS RESULTADOS */
SELECT V.NOME_COLABORADOR,
SUM(NVL(V.PLANO_SAUDE,0) + P.PS) AS TOTAL_PAGAR 
FROM VW_PAGAR_DEPENDENTE V 
INNER JOIN VW_PAGAR_SENIORIDADE P
ON V.MATRICULA = P.MATRICULA 
GROUP BY V.NOME_COLABORADOR
ORDER BY V.NOME_COLABORADOR;



/* PAGINAR LISTAGEM DE COLABORADORES */
SELECT * FROM (
SELECT ROWNUM AS LINHA, C.*
FROM BRH.COLABORADOR C
ORDER BY NOME)
--WHERE LINHA >=1 AND LINHA <=10 (1A PAGINA)
WHERE LINHA >= 11 AND LINHA <=20 -- (2A PAGINA)
-- WHERE LINHA >=21 AND LINHA <= 30 (3A PAGINA)
~~~
´´´

´´´ As principais procedures e functions
/* Criar procedure insere_projeto */

CREATE OR REPLACE PROCEDURE BRH.INSERE_PROJETO
(p_ID IN NUMBER, p_NOME IN VARCHAR2, p_RESPONSAVEL IN VARCHAR2, p_INICIO IN DATE)
IS
BEGIN
 INSERT INTO BRH.PROJETO (NOME, RESPONSAVEL, INICIO) VALUES (UPPER(p_NOME), p_RESPONSAVEL, p_INICIO);
END;   

--exemplo--
EXECUTE BRH.INSERE_PROJETO ('SPACLER', 'X123', '15/09/2022');
COMMIT;		



/* CRIAR FUNÇÃO CALCULA_IDADE */
	
CREATE OR REPLACE FUNCTION BRH.CALCULA_IDADE
(p_DATA_REFERENCIA %TYPE)
RETURN NUMBER
IS
    v_IDADE NUMBER;
BEGIN
    v_IDADE := FLOOR(MONTHS_BETWEEN (SYSDATE, p_DATA_REFERENCIA)/12);
    RETURN v_IDADE;
    END;
    --exemplo--
    SELECT BRH.CALCULA_IDADE (TO_DATE('11/03/2002', 'DD/MM/YYYY')) AS IDADE FROM DUAL;


/* EXERCÍCIO CRIAR FUNÇÃO FINALIZA_PROJETO */

CREATE OR REPLACE FUNCTION BRH.FINALIZA_PROJETO
(p_ID NUMBER)
RETURN BRH.PROJETO.FIM%TYPE
IS 
v_DATA_FIM PROJETO.FIM%TYPE;

BEGIN
v_DATA_FIM := SYSDATE;
    UPDATE BRH.PROJETO 
        SET FIM = v_DATA_FIM WHERE ID = p_ID;
            IF SQL%NOTFOUND THEN
               RAISE_APPLICATION_ERROR(-20000, 'PROJETO INEXISTENTE: ' || p_ID);
            END IF;
        
        RETURN v_DATA_FIM;
END;


/* VALIDAR NOVO PROJETO */
CREATE OR REPLACE PROCEDURE BRH.INSERE_PROJETO
(p_NOME IN VARCHAR2, p_RESPONSAVEL IN VARCHAR2, p_INICIO IN DATE)
IS
v_VALIDA_NOME NUMBER;
BEGIN
        v_VALIDA_NOME := LENGTH(p_NOME); 
        INSERT INTO BRH.PROJETO (NOME, RESPONSAVEL, INICIO) VALUES (UPPER(p_NOME), p_RESPONSAVEL, p_INICIO);
            IF v_VALIDA_NOME < 2 THEN 
                RAISE_APPLICATION_ERROR(-20942, 'Nome de projeto inválido! Deve ter dois ou mais caracteres.'  || p_NOME);
            END IF;
END; 
´´´

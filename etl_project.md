### Goal: select the contacts from 3 specific cities to lauch a marketing campaign and send to these contacts.

```
SET GLOBAL local_infile=1;

CREATE DATABASE 'database_name';

USE 'database_name';

-- I needed to work with 2 files that I named 'base_raw' and 'base_raw2', so the following processes were applied on both tables.
DROP TABLE IF EXISTS base_raw2;
CREATE TABLE base_raw2(
	`id_raw` VARCHAR(255),
    `id_cliente` VARCHAR(255),
    `cpf` VARCHAR(255),
    `nome` VARCHAR(255),
    `email` VARCHAR(255),
    `dt_nascimento` VARCHAR(255),
    `rg` VARCHAR(255),
    `genero` VARCHAR(255),
    `endereco` VARCHAR(255),
    `endereco_num` VARCHAR(255),
    `bairro` VARCHAR(255),
    `cidade` VARCHAR(255),
    `estado` VARCHAR(255),
    `cep` VARCHAR(255),
    `endereco_complemento` VARCHAR(255),
    `tel_prim` VARCHAR(255),
    `tel_2` VARCHAR(255),
    `envia_info` VARCHAR(255),
    `ip` VARCHAR(255),
    `dt_cadastro` VARCHAR(255),
    `dt_update` VARCHAR(255),
    `origem` VARCHAR(255),
    `plataforma` VARCHAR(255),
    `fb_id` VARCHAR(255),
    `promo_origem` VARCHAR(255)
);

-- loading the .csv file to the created tables
LOAD DATA LOCAL INFILE '/path/File.csv'
INTO TABLE base_raw2
FIELDS TERMINATED BY ';'
LINES TERMINATED BY '\r\n'
IGNORE 1 ROWS
(@Id, @IdCliente, @stCPF, @stNome, @stEmail, @dtNascimento, @stRG, @stGenero, @stRua, @stNum, @stBairro, @stCidade, @stEstado, @stCEP, @stComplemento, @stTel1, @stTel2, @btEnviaInformacoes, @stIP, @dtCadastro, @dtUpdate, @stOrigem, @stPlataforma, @stFBId, @stPromoOrigem)
SET
id_raw = nullif(@Id, ''),
id_cliente = nullif(@IdCliente, ''),
cpf = nullif(@stCPF, ''),
nome = nullif(@stNome, ''),
email = nullif(@stEmail, ''),
dt_nascimento = nullif(@dtNascimento, ''),
rg = nullif(@stRG, ''),
genero = nullif(@stGenero, ''),
endereco = nullif(@stRua, ''),
endereco_num = nullif(@stNum, ''),
bairro = nullif(@stBairro, ''),
cidade = nullif(@stCidade, ''),
estado = nullif(@stEstado, ''),
cep = nullif(@stCEP, ''),
endereco_complemento = nullif(@stComplemento, ''),
tel_prim = nullif(@stTel1, ''),
tel_2 = nullif(@stTel2, ''),
envia_info = nullif(@btEnviaInformacoes, ''),
ip = nullif(@stIP, ''),
dt_cadastro = nullif(@dtCadastro, ''),
dt_update = nullif(@dtUpdate, ''),
origem = nullif(@stOrigem, ''),
plataforma = nullif(@stPlataforma, ''),
fb_id = nullif(@stFBId, ''),
promo_origem = nullif(@stPromoOrigem, '')
;

-- some 'cpfs'(brazilian ID) had zeros on the left excluded so I needed to include them back 
BEGIN WORK;
ROLLBACK;
COMMIT;

UPDATE base_raw2
SET cpf = LPAD(cpf, 11, '0');

-- created a view with the union between both tables. I'm selecting only the cities we needed to see and every contact that allowed the contact(envia_info = 1)
DROP VIEW IF EXISTS union_promo;
CREATE VIEW union_promo AS
SELECT *
	FROM base_raw
	WHERE cidade IN ('São Paulo', 'Taboão da Serra', 'Embuguacu', 'Embu-guaçu') AND envia_info = 1
UNION
SELECT *
	FROM base_raw2
    WHERE cidade IN ('São Paulo', 'Taboão da Serra', 'Embuguacu', 'Embu-guaçu') AND envia_info = 1;

-- Creating the third table with additional information
CREATE TABLE cgs_ceps(
	`id_cg` VARCHAR(255),
    `contrato` VARCHAR(255),
    `nome` VARCHAR(255),
    `email` VARCHAR(255),
    `cpf` VARCHAR(255),
    `dt_nascimento` VARCHAR(255),
    `tel` VARCHAR(255),
    `cep_completo` VARCHAR(255),
    `cep` VARCHAR(255),
    `cidade` VARCHAR(255)
);

-- loading the .csv file to the created table
LOAD DATA LOCAL INFILE 'C:/path/file.csv'
INTO TABLE cgs_ceps
FIELDS TERMINATED BY ';'
LINES TERMINATED BY '\r\n'
IGNORE 1 ROWS
(@Id, @Contrato, @Nome, @Email, @Documento, @Nascimento, @Telefone, @cep_completo, @cep, @cidade)
SET
id_cg = nullif(@Id, ''),
contrato = nullif(@Contrato, ''),
nome = nullif(@Nome, ''),
email = nullif(@Email, ''),
cpf = nullif(@Documento, ''),
dt_nascimento = nullif(@Nascimento, ''),
tel = nullif(@Telefone, ''),
cep_completo = nullif(@cep_completo, ''),
cep = nullif(@cep, ''),
cidade = nullif(@cidade, '')
;

-- we had duplicates between this new table and the 'raw' ones. Since the third table had updated information on some contacts, I used join to bring these updated information.
CREATE VIEW cg_promo AS
SELECT up.*, cg.email AS email_atualizado, cg.cep AS cep_atualizado, cg.tel AS tel_atualizado
FROM union_promo up
	JOIN cgs_ceps cg
    ON up.cpf = cg.cpf;

-- created a fourth table with ceps(brazilian zip code) gathered with python automation using API requests from a public source
CREATE TABLE ceps(
	`cep` VARCHAR(255),
    `estado` VARCHAR(255),
    `cidade` VARCHAR(255),
    `bairro` VARCHAR(255),
    `rua` VARCHAR(255)
);

-- loading the .csv file to the created table
LOAD DATA LOCAL INFILE 'G:/path/file.csv'
INTO TABLE ceps
FIELDS TERMINATED BY ';'
LINES TERMINATED BY '\r\n'
IGNORE 1 ROWS
(@cep, @state, @city, @neighborhood, @street)
SET
cep = nullif(@cep, ''),
estado = nullif(@state, ''),
cidade = nullif(@city, ''),
bairro = nullif(@neighborhood, ''),
rua = nullif(@street, '')
;

```

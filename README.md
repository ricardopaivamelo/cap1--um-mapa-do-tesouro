# Modelo Entidade-Relacionamento (MER) - FarmTech Solutions

Este documento descreve o modelo conceitual e lógico do banco de dados para o sistema de monitoramento da FarmTech Solutions.

## 1. Entidades e Atributos

### 1.1. AreaPlantio
* **Propósito:** Representa as diferentes áreas ou talhões da plantação.
* **Atributos:**
    * `ID_AreaPlantio` (PK, Inteiro): Identificador único da área.
    * `NomeArea` (Texto/VARCHAR): Nome ou código da área.
    * `TamanhoArea` (Decimal, opcional): Tamanho em hectares.
    * `LocalizacaoDescricao` (Texto, opcional): Descrição da localização.

### 1.2. Cultura
* **Propósito:** Armazena informações sobre os tipos de culturas.
* **Atributos:**
    * `ID_Cultura` (PK, Inteiro): Identificador único da cultura.
    * `NomeCultura` (Texto/VARCHAR): Nome da cultura (ex: Milho).
    * `DescricaoCultura` (Texto, opcional): Detalhes sobre a cultura.

### 1.3. PlantioCultura
* **Propósito:** Associa Culturas a Areas de Plantio ao longo do tempo (Relação N:N).
* **Atributos:**
    * `ID_Plantio` (PK, Inteiro): Identificador único do registro de plantio.
    * `ID_AreaPlantio` (FK, Inteiro): Chave estrangeira referenciando `AreaPlantio`.
    * `ID_Cultura` (FK, Inteiro): Chave estrangeira referenciando `Cultura`.
    * `DataInicioPlantio` (Data/Timestamp): Início do plantio.
    * `DataFimPlantio` (Data/Timestamp, opcional): Fim do plantio/colheita.

### 1.4. Sensor
* **Propósito:** Informações sobre cada sensor físico.
* **Atributos:**
    * `ID_Sensor` (PK, Inteiro): Identificador único do sensor.
    * `TipoSensor` (Texto/VARCHAR): Tipo (Umidade, pH, Nutrientes P, Nutrientes K).
    * `ID_AreaPlantio` (FK, Inteiro): Chave estrangeira referenciando `AreaPlantio` onde está instalado.
    * `DataInstalacao` (Data, opcional): Data de instalação.
    * `StatusSensor` (Texto/VARCHAR, opcional): Ativo, Inativo, Manutenção.

### 1.5. LeituraSensor
* **Propósito:** Registros de dados coletados pelos sensores. 
* **Atributos:**
    * `ID_Leitura` (PK, Inteiro): Identificador único da leitura.
    * `ID_Sensor` (FK, Inteiro): Chave estrangeira referenciando `Sensor`. 
    * `DataHoraLeitura` (Timestamp): Momento da leitura. 
    * `ValorLeitura` (Decimal): Valor medido. 
    * `UnidadeMedida` (Texto/VARCHAR, opcional): Unidade (%, pH, ppm).

### 1.6. AjusteRecurso
* **Propósito:** Registra ações de irrigação ou aplicação de nutrientes. 
* **Atributos:**
    * `ID_Ajuste` (PK, Inteiro): Identificador único do ajuste.
    * `ID_AreaPlantio` (FK, Inteiro): Chave estrangeira referenciando `AreaPlantio`.
    * `TipoAjuste` (Texto/VARCHAR): 'Irrigacao' ou 'Nutriente'.
    * `DataHoraAjuste` (Timestamp): Momento do ajuste. 
    * `QuantidadeAplicada` (Decimal): Quantidade (litros, kg). 
    * `TipoNutriente` (Texto/VARCHAR, opcional): Qual nutriente (P, K, etc.). 
    * `ID_LeituraReferencia` (FK, Inteiro, opcional): Leitura que motivou o ajuste (referencia `LeituraSensor`).

## 2. Relacionamentos

* **AreaPlantio (1) : (N) Sensor:** Uma área pode ter vários sensores instalados.
    * *Implementação:* `Sensor.ID_AreaPlantio` (FK) referencia `AreaPlantio.ID_AreaPlantio`.
* **Sensor (1) : (N) LeituraSensor:** Um sensor pode realizar várias leituras.
    * *Implementação:* `LeituraSensor.ID_Sensor` (FK) referencia `Sensor.ID_Sensor`.
* **AreaPlantio (1) : (N) AjusteRecurso:** Uma área pode receber vários ajustes de recursos.
    * *Implementação:* `AjusteRecurso.ID_AreaPlantio` (FK) referencia `AreaPlantio.ID_AreaPlantio`.
* **AreaPlantio (N) : (N) Cultura (via PlantioCultura):** Uma área pode ter várias culturas ao longo do tempo, e uma cultura pode ser plantada em várias áreas.
    * *Implementação:* Tabela associativa `PlantioCultura` com FKs para `AreaPlantio` e `Cultura`.
        * `AreaPlantio` (1) : (N) `PlantioCultura`
        * `Cultura` (1) : (N) `PlantioCultura`
* **LeituraSensor (1) : (N) AjusteRecurso (Opcional):** Uma leitura pode (ou não) gerar um ou mais ajustes.
    * *Implementação:* `AjusteRecurso.ID_LeituraReferencia` (FK, permite nulos) referencia `LeituraSensor.ID_Leitura`.

## 3. Regras de Negócio (Exemplos)

* Toda leitura deve estar associada a um sensor existente.
* Todo sensor deve estar associado a uma área de plantio.
* Todo ajuste de recurso deve estar associado a uma área de plantio.
* Um registro em `PlantioCultura` representa uma cultura específica plantada em uma área específica durante um período.

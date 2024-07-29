## README - Transfomrnado Variaveis em Fator

### Visão Geral

Este script realiza manipulação e transformação de dados em um conjunto de dados lido de um arquivo Excel. O objetivo principal é criar um dicionário que inclua metadados sobre as variáveis do conjunto de dados, incluindo seus tipos, comprimentos de caracteres e níveis. O resultado final é salvo como um arquivo Excel.

### Requisitos

Certifique-se de ter os seguintes pacotes R instalados:
- `openxlsx`
- `dplyr`
- `tidyverse`
- `stringr`
- `tidyr`

Você pode instalar esses pacotes usando a função `install.packages`:

```R
install.packages(c('openxlsx', 'dplyr', 'tidyverse', 'stringr', 'tidyr'))
```

### Descrição do Script

1. **Carregando Pacotes:**
   ```R
   library('openxlsx')
   library('dplyr')
   library('tidyverse')
   library('stringr')
   library('tidyr')
   ```

2. **Lendo os Dados:**
   ```R
   df = read.xlsx("C:\\Users\\franc\\OneDrive\\Ãrea de Trabalho\\a.xlsx")
   ```

3. **Listando Variáveis e Seus Tipos:**
   A função `Dados` cria um data frame listando o índice, nome e classe de cada coluna.
   ```R
   Dados <- function(df) {
     data.frame(
       col_index = 1:ncol(df),
       col_name = colnames(df),
       col_class = sapply(df, class),
       row.names = NULL
     )
   }

   df1 <- Dados(df)
   ```

4. **Transformando Variáveis:**
   - **Caracter para Fator:**
     ```R
     for (i in 1:ncol(df)) {
       if (is.character(df[, i])) {
         df[, i] = factor(df[, i])
       }
     }
     ```

   - **Numérico para Fator:**
     ```R
     for (i in 1:ncol(df)) {
       if (is.numeric(df[, i])) {
         df[, i] = factor(df[, i])
       }
     }
     ```

5. **Extraindo Níveis de Cada Variável:**
   ```R
   df2 = sapply(df, levels)
   ```

6. **Transformando a Lista em Data Frame:**
   ```R
   df3 <- df2 %>%
     enframe() %>%
     unnest()
   ```

7. **Contando Caracteres por Nível:**
   ```R
   df3$noChar <- nchar(df3$value)
   ```

8. **Selecionando o Maior Número de Caracteres por Variável:**
   ```R
   df4 = df3 %>%
     group_by(name) %>%
     summarize(Estimativa_Caracteres = max(noChar, na.rm = TRUE)) %>%
     rename(col_name = name)
   ```

9. **Numerando Cada Nível:**
   ```R
   df3 = df3 %>%
     group_by(name) %>%
     mutate(id_nivel = row_number())
   ```

10. **Concatenando Identificadores aos Níveis:**
    ```R
    df3 <- df3 %>%
      mutate(value = paste(id_nivel, value, sep = " - "))
    ```

11. **Agrupando Níveis por Variável:**
    ```R
    df3 = df3 %>%
      group_by(name) %>%
      summarise(value = paste(value, collapse = " ; ")) %>%
      rename(col_name = name)
    ```

12. **Unificando Data Frames:**
    ```R
    df5 = left_join(df1, df4, by = "col_name")
    df_final = left_join(df5, df3, by = "col_name")
    ```

13. **Adicionando Campos Extras:**
    ```R
    df_final = df_final %>%
      add_column(hub = "",
                 produto = "",
                 planilha = "",
                 codigo = "",
                 casas_decimais = "",
                 requerido = "",
                 multipla_resposta = "",
                 descricao = "",
                 regra_validacao = "")
    ```

14. **Ordenando Campos:**
    ```R
    df_final = df_final %>%
      select(col_index, hub, produto, planilha, codigo, col_name, col_class,
             Estimativa_Caracteres, casas_decimais, requerido, multipla_resposta,
             descricao, regra_validacao, value)
    ```

15. **Salvando o Data Frame Final:**
    ```R
    write.xlsx(df_final, "C:\\Users\\franc\\OneDrive\\Area de Trabalho\\dicionario.xlsx")
    ```

### Saída

O script gerará um arquivo Excel chamado `dicionario.xlsx` na sua área de trabalho, contendo um dicionário detalhado do seu conjunto de dados com metadados e campos adicionais para uso posterior.

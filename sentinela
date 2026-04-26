#!/bin/bash

# =============== SISTEMAS OPERACIONAIS ===============
# Autor: Gabriel Rosa da Silva, RA 237069
# Repositório: ...
# =====================================================
# Script para monitorar de maneira inteligente o uso de 
# recursos e a integridade de diretórios críticos, 
# gerando alertas e tomando reações automáticas.
# Utiliza dos comandos utilitários 'log', 'slice' e
# 'trim', desenvolvidos externamente.

# Verificando se há privilégios necessários
if [ "$(whoami)" != "root" ]; then
   log -e "obrigatório executar com root"
   exit 1
fi

# Atribuindo usuário manualmente, pois executar o script
# com sudo torna USER=root, o que é indesejado dado o
# caminho da pasta de quarentena
USER="gabriel-silva"

# Declarando variáveis
quarentena="/home/$USER/quarentena"
temp="/tmp/sentinela"
logs="/var/log/log_sentinela.txt"
interrupt_solicited=0

# Interceptando sinal de interrupção (ctrl + C)
trap "interrupt_solicited=1" SIGINT

# Criando quarentena se não existir
if [ ! -d $quarentena ]; then
   mkdir $quarentena
fi

# Criando pasta temporária do processo.
# Neste caso serve apenas de exemplo para se fosse 
# necessário, visto que não é utilizada no script.
if [ ! -e $temp ]; then
   mkdir $temp
fi

# Executa as funções principais até receber uma interrupção
while [ $interrupt_solicited -eq 0 ]; do

   # Consulta 'ps' para o processo de maior consumo da CPU
   proc_details=$(ps -e -o pid,pcpu,ni,cmd --sort -pcpu | slice 2)

   # Recupera os dados desejados
   pid=$(           echo "$proc_details" | cut -c 1-7   | trim)
   uso_cpu=$(       echo "$proc_details" | cut -c 8-13  | trim)
   proc_niceness=$( echo "$proc_details" | cut -c 14-17 | trim)
   comando=$(       echo "$proc_details" | cut -c 18-   | trim)

   # Verifica se o processo consome mais de 80% da cpu
   # e se já não possui 'gentileza' 19.
   if [ $proc_niceness -ne 19 ]; then
      if awk -v n1="$uso_cpu" -v n2="80" 'BEGIN{exit !(n1 > n2)}' ; then
         # Altera gentileza e salva o log da ação
         renice 19 -p $pid | log -i | tee -a $logs
         log -w "Minimizando prioridade do processo pid:$pid, que estava consumindo ${uso_cpu}%% da CPU com comando '$comando'" | tee -a $logs
      fi
   fi

   # Procura por arquivos temporários maiores  que 5MB
   heavy_tmp_files=$(find /tmp -type f -size +5M | tr "\n" " ")
   if [ -n "$heavy_tmp_files" ]; then
      # Salva o log se existem e os move para quarentena
      log -w "Arquivos suspeitos encontrados e movidos para a quarentena: $heavy_tmp_files" | tee -a $logs
      mv $heavy_tmp_files $quarentena

      # Remove permissões de execução através de seus novos caminhos
      for file in $heavy_tmp_files; do
         chmod -x "$quarentena/$(basename $file)"
      done
   fi

   # Consulta uso atual de memória e número de processos em execução
   free_mem=$(free -m --si | slice 2 | cut -c 27-36 | trim)
   running_procs=$(ps -e | tail -n +2 | wc -l)

   # Salva os logs de status
   log -i "Uso de Memória Atual: ${free_mem}MB" | tee -a $logs
   log -i "Total de Processos em execução: ${running_procs}" | tee -a $logs
   log -i "Status do Sentinela: Ativo" | tee -a $logs

   # Aguarda 30 segundos antes de retornar ao início do loop
   sleep 30
done

# Processo interrompido, finalizando

# Limpando recursos utilizados, como a pasta temporária
rm -r $temp

# Exibindo e salvando log de encerramento
log -i "Sentinela encerrado com segurança" | tee -a $logs
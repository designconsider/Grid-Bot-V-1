# Grid-Bot-V-1
Robo em desenvolvimento para trade na B3, utilizando a pllataforma Meta trade
#property copyright "Copyright 2025"
#property link      "https:
#property version   "1.02"

// Inclusão da biblioteca padrão para operações de negociação
#include <Trade\Trade.mqh>  // Biblioteca para facilitar o envio de ordens no MQL5
CTrade trade;               // Declaração de um objeto "trade" para gerenciar ordens

// Parâmetros configuráveis pelo usuário
input int grid_count = 10;           // Número de níveis no grid (divisões no intervalo de preços)
input double min_price = 20.0;       // Preço mínimo do grid (limite inferior)
input double max_price = 25.0;       // Preço máximo do grid (limite superior)
input double tx_quantity = 1;      // Quantidade de ações ou lotes para cada transação
input double profit_percentage = 2.0;// Porcentagem de lucro desejado para as transações

//+------------------------------------------------------------------+
//| Função principal chamada em cada tick                            |
//+------------------------------------------------------------------+
void OnTick() {
   // Obtém o símbolo do ativo do gráfico em que o robô está rodando
   string symbol = Symbol();

   // Obtém o último preço de fechamento do ativo no período atual
   double last_price = iClose(symbol, PERIOD_CURRENT, 0);
   if (last_price == 0) {  // Verifica se o preço foi obtido com sucesso
      Print("Erro ao obter o preço do ativo: ", symbol); // Mensagem de erro no log
      return; // Sai da função se não conseguir obter o preço
   }

   // Calcula a largura de cada nível no grid (intervalo entre os preços)
   double grid_width = (max_price - min_price) / grid_count;

   // Itera pelos níveis de preços configurados no grid
   for (int i = 0; i < grid_count; i++) {
      double grid_price = min_price + (i * grid_width); // Calcula o preço de cada nível no grid

      // Verifica se o último preço está abaixo do nível atual do grid (oportunidade de compra)
      if (last_price < grid_price) {
         Print("Oportunidade de compra detectada no nível: ", grid_price); // Log da oportunidade de compra
         BuyOrder(symbol, tx_quantity, grid_price); // Envia uma ordem de compra
         break; // Sai do loop após encontrar uma oportunidade
      }
      // Verifica se o último preço está acima do nível atual do grid (oportunidade de venda)
      else if (last_price > grid_price) {
         Print("Oportunidade de venda detectada no nível: ", grid_price); // Log da oportunidade de venda
         SellOrder(symbol, tx_quantity, grid_price); // Envia uma ordem de venda
         break; // Sai do loop após encontrar uma oportunidade
      }
   }
}

//+------------------------------------------------------------------+
//| Função para enviar ordens de compra e armar venda pendente       |
//+------------------------------------------------------------------+
void BuyOrder(string symbol, double volume, double entry_price) {
   // Envia uma ordem de compra
   if (!trade.Buy(volume, symbol)) { // Verifica se a execução da ordem foi bem-sucedida
      Print("Erro ao enviar ordem de compra: ", trade.ResultRetcode()); // Mensagem de erro
   } else {
      Print("Compra executada com sucesso: ", trade.ResultOrder()); // Mensagem de sucesso
      // Calcula o preço-alvo para vender com o lucro desejado
      double target_price = entry_price * (1 + profit_percentage / 100);
      PlaceSellLimit(symbol, volume, target_price); // Arma uma ordem pendente de venda
   }
}

//+------------------------------------------------------------------+
//| Função para enviar ordens de venda e armar compra pendente       |
//+------------------------------------------------------------------+
void SellOrder(string symbol, double volume, double entry_price) {
   // Envia uma ordem de venda
   if (!trade.Sell(volume, symbol)) { // Verifica se a execução da ordem foi bem-sucedida
      Print("Erro ao enviar ordem de venda: ", trade.ResultRetcode()); // Mensagem de erro
   } else {
      Print("Venda executada com sucesso: ", trade.ResultOrder()); // Mensagem de sucesso
      // Calcula o preço-alvo para recompra com o lucro desejado
      double target_price = entry_price * (1 - profit_percentage / 100);
      PlaceBuyLimit(symbol, volume, target_price); // Arma uma ordem pendente de compra
   }
}

//+------------------------------------------------------------------+
//| Função para criar ordem pendente de venda                       |
//+------------------------------------------------------------------+
void PlaceSellLimit(string symbol, double volume, double target_price) {
   MqlTradeRequest request = {}; // Declara a estrutura para requisições de trade
   MqlTradeResult result = {};  // Declara a estrutura para resultados de trade

   // Preenche os detalhes da ordem pendente de venda
   request.action = TRADE_ACTION_PENDING; // Tipo de ação: Ordem pendente
   request.symbol = symbol;               // Símbolo do ativo
   request.volume = volume;               // Volume da transação
   request.type = ORDER_TYPE_SELL_LIMIT;  // Tipo da ordem pendente: Venda no limite
   request.price = target_price;          // Preço-alvo da venda
   request.deviation = 10;                // Desvio permitido para execução
   request.expiration = TimeCurrent() + 86400; // Tempo de expiração: 24 horas

   // Envia a ordem pendente de venda
   if (!OrderSend(request, result)) { // Verifica se houve erro no envio
      Print("Erro ao criar ordem pendente de venda: ", result.retcode); // Mensagem de erro
   } else {
      Print("Ordem pendente de venda criada com sucesso: ", result.order); // Mensagem de sucesso
   }
}

//+------------------------------------------------------------------+
//| Função para criar ordem pendente de compra                      |
//+------------------------------------------------------------------+
void PlaceBuyLimit(string symbol, double volume, double target_price) {
   MqlTradeRequest request = {}; // Declara a estrutura para requisições de trade
   MqlTradeResult result = {};  // Declara a estrutura para resultados de trade

   // Preenche os detalhes da ordem pendente de compra
   request.action = TRADE_ACTION_PENDING; // Tipo de ação: Ordem pendente
   request.symbol = symbol;               // Símbolo do ativo
   request.volume = volume;               // Volume da transação
   request.type = ORDER_TYPE_BUY_LIMIT;   // Tipo da ordem pendente: Compra no limite
   request.price = target_price;          // Preço-alvo da compra
   request.deviation = 10;                // Desvio permitido para execução
   request.expiration = TimeCurrent() + 86400; // Tempo de expiração: 24 horas

   // Envia a ordem pendente de compra
   if (!OrderSend(request, result)) { // Verifica se houve erro no envio
      Print("Erro ao criar ordem pendente de compra: ", result.retcode); // Mensagem de erro
   } else {
      Print("Ordem pendente de compra criada com sucesso: ", result.order); // Mensagem de sucesso
   }
}

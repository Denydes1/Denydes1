// Parámetros de la estrategia
input int short_sma_period = 10;   // Período de la media móvil corta
input int long_sma_period = 30;    // Período de la media móvil larga
input int rsi_period = 14;         // Período del RSI
input double rsi_overbought = 70;  // Nivel de sobrecompra del RSI
input double rsi_oversold = 30;    // Nivel de sobreventa del RSI
input double atr_multiplier = 1.5; // Multiplicador de ATR para Stop Loss y Take Profit
input double lot_size = 0.1;       // Tamaño del lote

//+------------------------------------------------------------------+
//| Script program start function                                    |
//+------------------------------------------------------------------+
void OnStart()
  {
   // Variables
   double short_sma, long_sma, rsi_value, atr_value;
   double price, stop_loss, take_profit;
   int ticket;
   double slippage = 5;   // Deslizamiento permitido en puntos
   int shift = 0;         // Barra actual

   // Obtenemos los valores de los indicadores
   short_sma = iMA(_Symbol, _Period, short_sma_period, 0, MODE_SMA, PRICE_CLOSE, shift);
   long_sma  = iMA(_Symbol, _Period, long_sma_period, 0, MODE_SMA, PRICE_CLOSE, shift);
   rsi_value = iRSI(_Symbol, _Period, rsi_period, PRICE_CLOSE, shift);
   atr_value = iATR(_Symbol, _Period, 14, shift); // Período estándar de 14 para el ATR

   price = SymbolInfoDouble(_Symbol, SYMBOL_BID); // Precio actual de oferta

   // Verificamos si ya hay alguna posición abierta
   if (PositionsTotal() > 0)
     {
      Print("Ya existe una posición abierta.");
      return;  // Salimos del script si ya hay una posición abierta
     }

   // Condición de Compra: Cruce SMA (corta > larga) + RSI en sobreventa
   if (short_sma > long_sma && rsi_value < rsi_oversold)
     {
      // Calcular Stop Loss y Take Profit basados en el ATR
      stop_loss = price - (atr_value * atr_multiplier);
      take_profit = price + (atr_value * atr_multiplier * 2); // Relación de TP:SL de 2:1

      // Abrir orden de compra
      ticket = trade.Buy(lot_size, _Symbol, price, stop_loss, take_profit);

      if (ticket == 0)
        {
         Print("Error al abrir la posición de compra: ", trade.ResultRetcode());
        }
      else
        {
         Print("Compra realizada con éxito, Ticket: ", ticket);
        }
     }

   // Condición de Venta: Cruce SMA (corta < larga) + RSI en sobrecompra
   if (short_sma < long_sma && rsi_value > rsi_overbought)
     {
      // Calcular Stop Loss y Take Profit basados en el ATR
      stop_loss = price + (atr_value * atr_multiplier);
      take_profit = price - (atr_value * atr_multiplier * 2); // Relación de TP:SL de 2:1

      // Abrir orden de venta
      ticket = trade.Sell(lot_size, _Symbol, price, stop_loss, take_profit);

      if (ticket == 0)
        {
         Print("Error al abrir la posición de venta: ", trade.ResultRetcode());
        }
      else
        {
         Print("Venta realizada con éxito, Ticket: ", ticket);
        }
     }

   Print("Operación finalizada.");
  }

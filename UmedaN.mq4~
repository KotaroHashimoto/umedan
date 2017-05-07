//+------------------------------------------------------------------+
//|                                                       UmedaN.mq4 |
//|                           Copyright 2017, Palawan Software, Ltd. |
//|                             https://coconala.com/services/204383 |
//+------------------------------------------------------------------+
#property copyright "Copyright 2017, Palawan Software, Ltd."
#property link      "https://coconala.com/services/204383"
#property description "Author: Kotaro Hashimoto <hasimoto.kotaro@gmail.com>"
#property version   "1.00"
#property strict

input int Magic_Number = 100; //マジックナンバー
input double Entry_Lot = 1.0; //エントリー総ロット数
input double Acceptable_Slippage = 1; //エントリー許容スリッページ (pips)
input double Acceptable_Spread = 5; //エントリー許容スプレッド (pips)
input int Take_Profit = 30; //非トレーリングストップ利確幅 (pips)
input int Trailing_Stop_Loss = 30; //トレーリングストップ追従幅 (pips)
input int TrailingSL_Percentage = 50; //全ポジションのうちのトレーリングストップの割合(%)
input int Determine_Duration = 10; //エントリー方向判定期間 (秒)
input int Entry_Threashould = 30; //エントリー方向判定用値動き閾値 (pips)
input int Time_Hour = 9;  //発動時間 (Hour)
input int Time_Minute = 30;  //発動時間 (Minute)

double minLot;
int lotCount;
datetime startTime;
double startPrice;
double tsl;

string lname = "lbl";

void drawLabel() {

  ObjectCreate(0, lname, OBJ_LABEL, 0, 0, 0);
  ObjectSetInteger(0, lname, OBJPROP_CORNER, CORNER_LEFT_UPPER);
  ObjectSet(lname, OBJPROP_TIMEFRAMES, OBJ_ALL_PERIODS);
  ObjectSetInteger(0, lname, OBJPROP_SELECTABLE, false);

  ObjectSetText(lname, "", 16, "Arial", clrYellow);
}  
  
//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
int OnInit()
  {
//---
   
   minLot = MarketInfo(Symbol(), MODE_MINLOT);
   startTime = -1;
   lotCount = -1;
   startPrice = -1.0;
   tsl = NormalizeDouble(Trailing_Stop_Loss * Point * 10.0, Digits);
   
   drawLabel();
   
//---
   return(INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Expert deinitialization function                                 |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
//---
  ObjectDelete(0, lname);
   
  }
//+------------------------------------------------------------------+
//| Expert tick function                                             |
//+------------------------------------------------------------------+
void OnTick()
  {
//---
  datetime tl = TimeLocal();
  

  if(TimeHour(tl) < Time_Hour || TimeMinute(tl) < Time_Minute) {
  
    int t = 360 * (Time_Hour - TimeHour(tl)) + 60 * (Time_Minute - TimeMinute(tl)) + (0 - TimeSeconds(tl) % 60);
    ObjectSetText(lname, IntegerToString(t), 16, "Arial", clrYellow);
    return;
  }
  else if(startTime < 0) {
    ObjectSetText(lname, "0", 16, "Arial", clrYellow);
    startTime = tl;
  }

  if(tl - startTime < Determine_Duration) {
    ObjectSetText(lname, IntegerToString(startTime - TimeCurrent()), 16, "Arial", clrYellow);
    return;
  }
  else if(startPrice < 0.0){
    ObjectSetText(lname, "Determining Direction ...", 16, "Arial", clrYellow);
    startPrice = (Ask + Bid) / 2.0;
  }

  if(10 * Acceptable_Spread < MarketInfo(Symbol(), MODE_SPREAD) || Acceptable_Spread <= 0.0) {
    ObjectSetText(lname, "Waiting for the spread settled ...", 16, "Arial", clrYellow);
    return;
  }
  
  
  for(int i = 0; i < OrdersTotal(); i++) {
    if(OrderSelect(i, SELECT_BY_POS)) {
      if(OrderMagicNumber() == Magic_Number && OrderTakeProfit() == 0) {
        if(OrderType() == OP_BUY) {
          if(OrderStopLoss() + tsl < Ask) {
            bool mod = OrderModify(OrderTicket(), OrderOpenPrice(), Ask - tsl, 0, 0);
          }
        }
        else {
          if(Bid + tsl < OrderStopLoss()) {
            bool mod = OrderModify(OrderTicket(), OrderOpenPrice(), Bid + tsl, 0, 0);
          }
        }
      }
    }
  }
  
  
  if(lotCount < 0) {
  
    lotCount = int(MathFloor(Entry_Lot / minLot));
    double th = Entry_Threashould + Point * 10.0;
  
    int cmd = -1;
    double price = (Ask + Bid) / 2.0;
    double tp;
    double sl;
    
    if(startPrice + th < price) {
      cmd = OP_BUY;
      price = NormalizeDouble(Ask, Digits);
      tp = NormalizeDouble(Ask + Take_Profit * Point * 10.0, Digits);
      sl = NormalizeDouble(Ask - tsl, Digits);
      ObjectSetText(lname, "Long Entry", 16, "Arial", clrYellow);
    }
    else if(price + th < startPrice) {
      cmd = OP_SELL;
      price = NormalizeDouble(Bid, Digits);
      tp = NormalizeDouble(Bid - Take_Profit * Point * 10.0, Digits);
      sl = NormalizeDouble(Bid + tsl, Digits);
      ObjectSetText(lname, "Short Entry", 16, "Arial", clrYellow);
    }
    else {
      ObjectSetText(lname, "Insufficient Movement. No Entry.", 16, "Arial", clrYellow);
      Print("Didn't reach price threashould(", th, ") : ", MathAbs(startPrice - price));
      return;
    }
  
    if(100 < TrailingSL_Percentage) {
      Print("Invalid Parameter TrailingSL_Percentage(< 100): ", TrailingSL_Percentage);
      return;
    }
    
    int trailPos = int(MathFloor(lotCount * TrailingSL_Percentage / 100.0));
    int tpPos = lotCount - trailPos;
    string symbol = Symbol();
    int slip = int(Acceptable_Slippage * 10.0);
    int spread = int(Acceptable_Spread * 10.0);

    
    for(int i = 0; i < tpPos; i++) {
      int ticket = OrderSend(symbol, cmd, minLot, price, slip, sl, tp, NULL, Magic_Number);
    }
    for(int i = 0; i < trailPos; i++) {
      int ticket = OrderSend(symbol, cmd, minLot, price, slip, sl, 0, NULL, Magic_Number);
    }
  }
}
//+------------------------------------------------------------------+

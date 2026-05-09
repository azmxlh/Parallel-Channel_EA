//+------------------------------------------------------------------+
//|  Parallel Channel EA — Full MT5 Auto Trading Version            |
//|  Converted from Pine Script v6                                   |
//|  Includes: SuperTrend + Channel Breakout + MTF Dashboard        |
//|  Auto Trade: Standard Account, Risk % based position sizing     |
//+------------------------------------------------------------------+
#property copyright   "Parallel Channel EA"
#property version     "1.00"
#property strict

#include <Trade\Trade.mqh>
#include <Trade\PositionInfo.mqh>

CTrade        trade;
CPositionInfo posInfo;

//+------------------------------------------------------------------+
//| ENUMS                                                            |
//+------------------------------------------------------------------+
enum ENUM_DASH_POS
  {
   DASH_TOP_RIGHT    = 0, // Top Right
   DASH_TOP_LEFT     = 1, // Top Left
   DASH_BOTTOM_RIGHT = 2, // Bottom Right
   DASH_BOTTOM_LEFT  = 3  // Bottom Left
  };

//+------------------------------------------------------------------+
//| INPUT PARAMETERS                                                 |
//+------------------------------------------------------------------+

// ── Pivot Settings ──────────────────────────────────────────────
input group            "══ Pivot Settings ══"
input int              PivotLen        = 10;     // Pivot Length (Left/Right)
input int              MaxStorePvt     = 40;     // Max Stored Pivots

// ── Channel Settings ────────────────────────────────────────────
input group            "══ Channel Settings ══"
input int              ChMinPivots     = 3;      // Min Pivots to Form Channel
input int              ChMinBars       = 5;      // Min Channel Bars
input int              ChMaxBars       = 500;    // Max Channel Bars
input int              ChATRLen        = 14;     // ATR Length
input double           ChQuality       = 0.35;   // Min Channel Quality
input double           ChPivotTol      = 0.5;    // Pivot Proximity (xATR)
input bool             ChShowMid       = true;   // Show Midline
input bool             ChExtend        = true;   // Extend Channel Right
input color            ChUpCol         = clrLime;     // Up Channel Color
input color            ChDnCol         = clrRed;      // Down Channel Color

// ── SuperTrend Settings ─────────────────────────────────────────
input group            "══ SuperTrend Settings ══"
input int              StATRLen        = 10;     // ST ATR Length
input double           StFactor        = 3.0;    // ST Factor
input bool             StShowLine      = false;  // Show SuperTrend Line
input color            StBullCol       = clrLime;     // Bull Color
input color            StBearCol       = clrRed;      // Bear Color

// ── EMA Confluence Filter ───────────────────────────────────────
input group            "══ EMA Confluence Filter ══"
input bool             EfEnabled       = true;   // Enable EMA Filter
input int              EfEMA1          = 100;    // EMA 1 Length
input int              EfEMA2          = 200;    // EMA 2 Length
input bool             EfShowEMA       = false;  // Show EMA Lines

// ── Signal Settings ─────────────────────────────────────────────
input group            "══ Signal Settings ══"
input bool             SigEnabled      = true;   // Enable ST Flip Signals
input int              SigCooldown     = 3;      // Signal Cooldown (bars)
input bool             BoUpEnabled     = true;   // Enable Up Channel Breakout
input int              BoUpConfirm     = 1;      // Up CH Confirm Bars
input int              BoUpCooldown    = 5;      // Up CH Cooldown
input bool             BoDnEnabled     = true;   // Enable Down Channel Breakout
input int              BoDnConfirm     = 1;      // Dn CH Confirm Bars
input int              BoDnCooldown    = 5;      // Dn CH Cooldown

// ── Risk Management ─────────────────────────────────────────────
input group            "══ Risk Management ══"
input double           RiskPercent     = 1.0;    // Risk % per Trade
input double           SlATRMult       = 1.5;    // SL ATR Multiplier
input double           SlRR            = 2.0;    // Risk:Reward Ratio
input bool             CloseOnOpposite = true;   // Close on Opposite Signal

// ── MTF Dashboard ───────────────────────────────────────────────
input group            "══ MTF Dashboard ══"
input bool             ShowDash        = true;   // Show Dashboard
input ENUM_DASH_POS    DashPosition    = DASH_TOP_RIGHT; // Dashboard Position
input int              DashEMA1        = 50;     // Dashboard EMA Fast
input int              DashEMA2        = 100;    // Dashboard EMA Mid
input int              DashEMA3        = 200;    // Dashboard EMA Slow
input int              AutoMinTFs      = 3;      // Min TFs Agreeing (1-6)

//+------------------------------------------------------------------+
//| GLOBAL VARIABLES                                                 |
//+------------------------------------------------------------------+

// ATR
double atrVal = 0;

// SuperTrend state
double stValue       = 0;
bool   stBull        = false;
bool   stBullPrev    = false;
bool   stFlipBull    = false;
bool   stFlipBear    = false;

// EMA
double ema1Val = 0, ema2Val = 0;
bool   emaAllAbove = false, emaAllBelow = false;

// Pivot arrays
double hiP[];  int hiX[];
double loP[];  int loX[];
int    hiCount = 0, loCount = 0;

// Channel Up state
int    chUX1 = -1; double chUY1 = 0;
int    chUX2 = -1; double chUY2 = 0;
double chUOff = 0; bool   chUOn = false; int chUPvts = 0;

// Channel Down state
int    chDX1 = -1; double chDY1 = 0;
int    chDX2 = -1; double chDY2 = 0;
double chDOff = 0; bool   chDOn = false; int chDPvts = 0;

// Breakout counters
int    upAboveCount = 0, upBelowCount = 0;
int    dnAboveCount = 0, dnBelowCount = 0;
double upChUpper = 0, upChLower = 0;
double dnChUpper = 0, dnChLower = 0;

// Cooldown tracking
int    lastUpBOBar  = -999;
int    lastDnBOBar  = -999;
int    lastSTBuyBar = -999;
int    lastSTSellBar= -999;

// Signal flags
bool   sigBuy  = false, sigSell  = false;
bool   upBOBuy = false, upBOSell = false;
bool   dnBOBuy = false, dnBOSell = false;
bool   doLong  = false, doShort  = false;

// MTF scores
int    s5=0,  s15=0,  s30=0,  s60=0,  s240=0,  sD=0;
bool   a5_1=false,  a5_2=false,  a5_3=false;
bool   a15_1=false, a15_2=false, a15_3=false;
bool   a30_1=false, a30_2=false, a30_3=false;
bool   a60_1=false, a60_2=false, a60_3=false;
bool   a240_1=false,a240_2=false,a240_3=false;
bool   aD_1=false,  aD_2=false,  aD_3=false;
int    mtfBullCount=0, mtfBearCount=0;
int    cMtfDir=0;
bool   allowBuy=false, allowSell=false;

// Active position direction for dashboard
int    activeSigDir = 0; // 1=long, -1=short, 0=flat

// Previous bar index for bar-close detection
int    prevBarIndex = -1;

// Dashboard object prefix
string OBJ_PREFIX = "PCEA_";

//+------------------------------------------------------------------+
//| OnInit                                                           |
//+------------------------------------------------------------------+
int OnInit()
  {
   ArrayResize(hiP, MaxStorePvt);
   ArrayResize(hiX, MaxStorePvt);
   ArrayResize(loP, MaxStorePvt);
   ArrayResize(loX, MaxStorePvt);
   hiCount = 0;
   loCount = 0;

   trade.SetExpertMagicNumber(202406);
   trade.SetDeviationInPoints(10);
   trade.SetTypeFilling(ORDER_FILLING_IOC);

   // Clean old dashboard objects
   DeleteDashboard();

   Print("Parallel Channel EA initialized on ", Symbol(), " ", EnumToString(Period()));
   return INIT_SUCCEEDED;
  }

//+------------------------------------------------------------------+
//| OnDeinit                                                         |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
   DeleteDashboard();
   DeleteChannelObjects();
   Comment("");
  }

//+------------------------------------------------------------------+
//| OnTick — main entry                                             |
//+------------------------------------------------------------------+
void OnTick()
  {
   // Only process on new confirmed bar
   int currentBar = iBars(Symbol(), Period()) - 1;
   if(currentBar == prevBarIndex)
      return;
   prevBarIndex = currentBar;

   // Run all calculations on bar close
   CalculateATR();
   CalculateSuperTrend();
   CalculateEMA();
   CalculateMTFBias();
   DetectPivots();
   FindChannels();
   DrawChannels();
   CalculateBreakouts();
   GenerateSignals();
   ExecuteTrades();

   if(ShowDash)
      DrawDashboard();
  }

//+------------------------------------------------------------------+
//| ATR Calculation                                                  |
//+------------------------------------------------------------------+
void CalculateATR()
  {
   double atrBuf[];
   ArraySetAsSeries(atrBuf, true);
   int handle = iATR(Symbol(), Period(), ChATRLen);
   if(handle == INVALID_HANDLE) return;
   CopyBuffer(handle, 0, 1, 1, atrBuf);
   atrVal = atrBuf[0];
   IndicatorRelease(handle);
  }

//+------------------------------------------------------------------+
//| SuperTrend Calculation                                          |
//| Converted from Pine Script ta.supertrend()                      |
//+------------------------------------------------------------------+
void CalculateSuperTrend()
  {
   int lookback = StATRLen * 3 + 10;
   double highArr[], lowArr[], closeArr[], atrArr[];
   ArraySetAsSeries(highArr,  true);
   ArraySetAsSeries(lowArr,   true);
   ArraySetAsSeries(closeArr, true);
   ArraySetAsSeries(atrArr,   true);

   CopyHigh(Symbol(),  Period(), 0, lookback, highArr);
   CopyLow(Symbol(),   Period(), 0, lookback, lowArr);
   CopyClose(Symbol(), Period(), 0, lookback, closeArr);

   int atrHandle = iATR(Symbol(), Period(), StATRLen);
   if(atrHandle == INVALID_HANDLE) return;
   CopyBuffer(atrHandle, 0, 0, lookback, atrArr);
   IndicatorRelease(atrHandle);

   // Calculate SuperTrend from scratch for lookback bars
   double upperBand = 0, lowerBand = 0;
   double prevUpper = 0, prevLower = 0;
   bool   prevBull  = true;
   double prevST    = 0;

   for(int i = lookback - 1; i >= 1; i--)
     {
      double hl2    = (highArr[i] + lowArr[i]) / 2.0;
      double atr_i  = atrArr[i];
      double bUp    = hl2 + StFactor * atr_i;
      double bDn    = hl2 - StFactor * atr_i;

      // Band adjustment logic
      upperBand = (bUp < prevUpper || closeArr[i+1] > prevUpper) ? bUp : prevUpper;
      lowerBand = (bDn > prevLower || closeArr[i+1] < prevLower) ? bDn : prevLower;

      bool isBull;
      if(prevST == prevUpper)
         isBull = closeArr[i] > upperBand;
      else
         isBull = closeArr[i] >= lowerBand;

      stValue   = isBull ? lowerBand : upperBand;
      prevUpper = upperBand;
      prevLower = lowerBand;
      prevBull  = isBull;
      prevST    = stValue;

      // At bar index 1 (previous confirmed bar) capture state
      if(i == 1)
        {
         stBullPrev = isBull;
        }
      // At bar index 0 logic done below
     }

   // Current bar (bar 0) — use confirmed close from bar 1
   double hl2_0   = (highArr[0] + lowArr[0]) / 2.0;
   double bUp0    = hl2_0 + StFactor * atrArr[0];
   double bDn0    = hl2_0 - StFactor * atrArr[0];
   upperBand      = (bUp0 < prevUpper || closeArr[1] > prevUpper) ? bUp0 : prevUpper;
   lowerBand      = (bDn0 > prevLower || closeArr[1] < prevLower) ? bDn0 : prevLower;

   bool isBullNow;
   if(prevST == prevUpper)
      isBullNow = closeArr[0] > upperBand;
   else
      isBullNow = closeArr[0] >= lowerBand;

   stBull     = stBullPrev; // Use confirmed previous bar
   stFlipBull = stBull  && !stBullPrev;
   stFlipBear = !stBull && stBullPrev;
  }

//+------------------------------------------------------------------+
//| EMA Calculation                                                  |
//+------------------------------------------------------------------+
void CalculateEMA()
  {
   double e1[], e2[], cl[];
   ArraySetAsSeries(e1, true);
   ArraySetAsSeries(e2, true);
   ArraySetAsSeries(cl, true);

   int h1 = iMA(Symbol(), Period(), EfEMA1, 0, MODE_EMA, PRICE_CLOSE);
   int h2 = iMA(Symbol(), Period(), EfEMA2, 0, MODE_EMA, PRICE_CLOSE);
   if(h1 == INVALID_HANDLE || h2 == INVALID_HANDLE) return;

   CopyBuffer(h1, 0, 1, 1, e1);
   CopyBuffer(h2, 0, 1, 1, e2);
   CopyClose(Symbol(), Period(), 1, 1, cl);

   ema1Val    = e1[0];
   ema2Val    = e2[0];
   emaAllAbove = cl[0] > ema1Val && cl[0] > ema2Val;
   emaAllBelow = cl[0] < ema1Val && cl[0] < ema2Val;

   IndicatorRelease(h1);
   IndicatorRelease(h2);

   // Draw EMA lines if enabled
   if(EfShowEMA)
     {
      DrawHLine(OBJ_PREFIX + "EMA1", ema1Val, clrDodgerBlue, 1, STYLE_SOLID);
      DrawHLine(OBJ_PREFIX + "EMA2", ema2Val, clrOrange,     1, STYLE_SOLID);
     }
  }

//+------------------------------------------------------------------+
//| MTF Bias Calculation                                             |
//| Equivalent to Pine Script f_bias() with request.security()      |
//+------------------------------------------------------------------+
void CalculateMTFBias()
  {
   ENUM_TIMEFRAMES tfEnum[6] = {PERIOD_M5, PERIOD_M15, PERIOD_M30,
                                 PERIOD_H1, PERIOD_H4,  PERIOD_D1};
   int scores[6];
   bool arr1[6], arr2[6], arr3[6];

   for(int t = 0; t < 6; t++)
     {
      double c[], e1[], e2[], e3[];
      ArraySetAsSeries(c,  true);
      ArraySetAsSeries(e1, true);
      ArraySetAsSeries(e2, true);
      ArraySetAsSeries(e3, true);

      int hC  = iClose_MTF(Symbol(), tfEnum[t], 2, c);
      int hE1 = iMA(Symbol(), tfEnum[t], DashEMA1, 0, MODE_EMA, PRICE_CLOSE);
      int hE2 = iMA(Symbol(), tfEnum[t], DashEMA2, 0, MODE_EMA, PRICE_CLOSE);
      int hE3 = iMA(Symbol(), tfEnum[t], DashEMA3, 0, MODE_EMA, PRICE_CLOSE);

      if(hE1 == INVALID_HANDLE || hE2 == INVALID_HANDLE || hE3 == INVALID_HANDLE)
        {
         scores[t] = 0; arr1[t] = false; arr2[t] = false; arr3[t] = false;
         continue;
        }

      CopyBuffer(hE1, 0, 1, 1, e1);
      CopyBuffer(hE2, 0, 1, 1, e2);
      CopyBuffer(hE3, 0, 1, 1, e3);

      double closeMTF = c[0];
      bool   b1 = closeMTF > e1[0];
      bool   b2 = closeMTF > e2[0];
      bool   b3 = closeMTF > e3[0];

      scores[t] = (b1 ? 1 : -1) + (b2 ? 1 : -1) + (b3 ? 1 : -1);
      arr1[t] = b1; arr2[t] = b2; arr3[t] = b3;

      IndicatorRelease(hE1);
      IndicatorRelease(hE2);
      IndicatorRelease(hE3);
     }

   s5   = scores[0]; a5_1  = arr1[0]; a5_2  = arr2[0]; a5_3  = arr3[0];
   s15  = scores[1]; a15_1 = arr1[1]; a15_2 = arr2[1]; a15_3 = arr3[1];
   s30  = scores[2]; a30_1 = arr1[2]; a30_2 = arr2[2]; a30_3 = arr3[2];
   s60  = scores[3]; a60_1 = arr1[3]; a60_2 = arr2[3]; a60_3 = arr3[3];
   s240 = scores[4]; a240_1= arr1[4]; a240_2= arr2[4]; a240_3= arr3[4];
   sD   = scores[5]; aD_1  = arr1[5]; aD_2  = arr2[5]; aD_3  = arr3[5];

   mtfBullCount = (s5>0?1:0)+(s15>0?1:0)+(s30>0?1:0)+(s60>0?1:0)+(s240>0?1:0)+(sD>0?1:0);
   mtfBearCount = (s5<0?1:0)+(s15<0?1:0)+(s30<0?1:0)+(s60<0?1:0)+(s240<0?1:0)+(sD<0?1:0);

   cMtfDir  = (mtfBullCount >= AutoMinTFs) ? 1 : (mtfBearCount >= AutoMinTFs) ? -1 : 0;
   allowBuy  = (cMtfDir == 1);
   allowSell = (cMtfDir == -1);
  }

//+------------------------------------------------------------------+
//| Helper: Get MTF close price                                      |
//+------------------------------------------------------------------+
int iClose_MTF(string symbol, ENUM_TIMEFRAMES tf, int count, double &arr[])
  {
   return CopyClose(symbol, tf, 1, count, arr);
  }

//+------------------------------------------------------------------+
//| Pivot Detection                                                  |
//| Equivalent to ta.pivothigh / ta.pivotlow                        |
//+------------------------------------------------------------------+
void DetectPivots()
  {
   int totalBars = iBars(Symbol(), Period());
   if(totalBars < PivotLen * 2 + 2) return;

   double hi[], lo[];
   ArraySetAsSeries(hi, true);
   ArraySetAsSeries(lo, true);
   CopyHigh(Symbol(), Period(), 0, PivotLen * 2 + 3, hi);
   CopyLow(Symbol(),  Period(), 0, PivotLen * 2 + 3, lo);

   // Check pivot high at bar [PivotLen+1] (confirmed)
   int pvtBar = PivotLen + 1;
   bool isPivotHigh = true;
   bool isPivotLow  = true;

   for(int k = 0; k <= PivotLen * 2; k++)
     {
      if(k == pvtBar) continue;
      if(hi[k] >= hi[pvtBar]) isPivotHigh = false;
      if(lo[k] <= lo[pvtBar]) isPivotLow  = false;
     }

   int barIdx = totalBars - 1 - pvtBar;

   if(isPivotHigh)
     {
      // Check if already stored
      bool exists = false;
      for(int i = 0; i < hiCount; i++)
         if(hiX[i] == barIdx) { exists = true; break; }

      if(!exists)
        {
         if(hiCount < MaxStorePvt)
           {
            hiP[hiCount] = hi[pvtBar];
            hiX[hiCount] = barIdx;
            hiCount++;
           }
         else
           {
            // Shift array left and add new
            for(int i = 0; i < MaxStorePvt - 1; i++)
              { hiP[i] = hiP[i+1]; hiX[i] = hiX[i+1]; }
            hiP[MaxStorePvt-1] = hi[pvtBar];
            hiX[MaxStorePvt-1] = barIdx;
           }
        }
     }

   if(isPivotLow)
     {
      bool exists = false;
      for(int i = 0; i < loCount; i++)
         if(loX[i] == barIdx) { exists = true; break; }

      if(!exists)
        {
         if(loCount < MaxStorePvt)
           {
            loP[loCount] = lo[pvtBar];
            loX[loCount] = barIdx;
            loCount++;
           }
         else
           {
            for(int i = 0; i < MaxStorePvt - 1; i++)
              { loP[i] = loP[i+1]; loX[i] = loX[i+1]; }
            loP[MaxStorePvt-1] = lo[pvtBar];
            loX[MaxStorePvt-1] = barIdx;
           }
        }
     }
  }

//+------------------------------------------------------------------+
//| Project price along a line at given bar index                   |
//| Equivalent to Pine Script linePrice()                           |
//+------------------------------------------------------------------+
double LinePrice(int x1, double y1, int x2, double y2, int bx)
  {
   if(x2 == x1) return y1;
   return y1 + (y2 - y1) / (double)(x2 - x1) * (bx - x1);
  }

//+------------------------------------------------------------------+
//| Count pivots within tolerance of a trendline                    |
//+------------------------------------------------------------------+
int CountPivotsOnLine(double &pP[], int &pX[], int count,
                      int x1, double y1, int x2, double y2,
                      double tol, int currentBar)
  {
   int cnt = 0;
   for(int i = 0; i < count; i++)
     {
      if(pX[i] >= x1 && pX[i] <= currentBar)
        {
         double expected = LinePrice(x1, y1, x2, y2, pX[i]);
         if(MathAbs(pP[i] - expected) <= tol)
            cnt++;
        }
     }
   return cnt;
  }

//+------------------------------------------------------------------+
//| Find Best Channel                                                |
//| Equivalent to Pine Script findChannel()                         |
//+------------------------------------------------------------------+
bool FindBestChannel(double &bP[], int &bX[], int bCount,
                     double &oP[], int &oX[], int oCount,
                     bool isUp, int currentBar,
                     int &outX1, double &outY1,
                     int &outX2, double &outY2,
                     double &outOff, int &outPvts)
  {
   outX1 = -1; outY1 = 0; outX2 = -1; outY2 = 0;
   outOff = 0; outPvts = 0;
   double bestQ = 0.0;

   if(bCount < 2) return false;

   int pairs = MathMin(bCount, 8);
   double tol = atrVal * ChPivotTol;

   for(int i = bCount - 1; i >= MathMax(0, bCount - pairs); i--)
     {
      for(int j = i - 1; j >= MathMax(0, i - 5); j--)
        {
         int    ix1 = bX[j]; double iy1 = bP[j];
         int    ix2 = bX[i]; double iy2 = bP[i];
         int    span = ix2 - ix1;

         if(span < ChMinBars || span > ChMaxBars)      continue;
         if(isUp  && iy2 <= iy1)                        continue;
         if(!isUp && iy2 >= iy1)                        continue;

         int pvtOnBase = CountPivotsOnLine(bP, bX, bCount, ix1, iy1, ix2, iy2, tol, currentBar);
         if(pvtOnBase < ChMinPivots) continue;

         double extremeOff = 0.0;
         for(int k = 0; k < oCount; k++)
           {
            if(oX[k] >= ix1 && oX[k] <= currentBar)
              {
               double d = oP[k] - LinePrice(ix1, iy1, ix2, iy2, oX[k]);
               if(isUp  && d > extremeOff) extremeOff = d;
               if(!isUp && d < extremeOff) extremeOff = d;
              }
           }

         if(isUp  && extremeOff <= 0) continue;
         if(!isUp && extremeOff >= 0) continue;

         int   totalB   = currentBar - ix1;
         if(totalB <= 0) continue;
         int   checkLen = MathMin(totalB, 200);
         int   contained = 0;
         double qTol    = atrVal * 0.05;

         double cl[];
         ArraySetAsSeries(cl, true);
         double hh[], ll[];
         ArraySetAsSeries(hh, true);
         ArraySetAsSeries(ll, true);
         CopyClose(Symbol(), Period(), 0, checkLen + 2, cl);
         CopyHigh(Symbol(),  Period(), 0, checkLen + 2, hh);
         CopyLow(Symbol(),   Period(), 0, checkLen + 2, ll);

         for(int k = 0; k < checkLen; k++)
           {
            int bx = currentBar - k;
            if(bx < ix1) break;
            double bLine = LinePrice(ix1, iy1, ix2, iy2, bx);
            if(isUp)
              { if(ll[k] >= bLine - qTol && hh[k] <= bLine + extremeOff + qTol) contained++; }
            else
              { if(hh[k] <= bLine + qTol && ll[k] >= bLine + extremeOff - qTol) contained++; }
           }

         double q     = checkLen > 0 ? (double)contained / (double)checkLen : 0.0;
         double score = q + (double)pvtOnBase * 0.05;

         if(q >= ChQuality && score > bestQ + (double)outPvts * 0.05)
           {
            bestQ   = q;
            outX1   = ix1; outY1 = iy1;
            outX2   = ix2; outY2 = iy2;
            outOff  = extremeOff;
            outPvts = pvtOnBase;
           }
        }
     }

   return outX1 != -1;
  }

//+------------------------------------------------------------------+
//| Find Channels — Up and Down                                     |
//+------------------------------------------------------------------+
void FindChannels()
  {
   int currentBar = iBars(Symbol(), Period()) - 1;

   if(loCount >= 2)
     {
      int nx1; double ny1; int nx2; double ny2; double no; int npc;
      if(FindBestChannel(loP, loX, loCount, hiP, hiX, hiCount, true,
                         currentBar, nx1, ny1, nx2, ny2, no, npc))
        {
         chUX1=nx1; chUY1=ny1; chUX2=nx2; chUY2=ny2;
         chUOff=no; chUOn=true; chUPvts=npc;
        }
     }

   if(hiCount >= 2)
     {
      int nx1; double ny1; int nx2; double ny2; double no; int npc;
      if(FindBestChannel(hiP, hiX, hiCount, loP, loX, loCount, false,
                         currentBar, nx1, ny1, nx2, ny2, no, npc))
        {
         chDX1=nx1; chDY1=ny1; chDX2=nx2; chDY2=ny2;
         chDOff=no; chDOn=true; chDPvts=npc;
        }
     }
  }

//+------------------------------------------------------------------+
//| Draw Channel Lines on Chart                                      |
//+------------------------------------------------------------------+
void DrawChannels()
  {
   int totalBars = iBars(Symbol(), Period());
   datetime t1, t2;

   if(chUOn && chUX1 >= 0)
     {
      t1 = iTime(Symbol(), Period(), totalBars - 1 - chUX1);
      t2 = iTime(Symbol(), Period(), totalBars - 1 - chUX2);

      DrawTrendLine(OBJ_PREFIX+"UBase", t1, chUY1, t2, chUY2, ChUpCol, 2, ChExtend);
      DrawTrendLine(OBJ_PREFIX+"UPar",  t1, chUY1+chUOff, t2, chUY2+chUOff, ChUpCol, 2, ChExtend);
      if(ChShowMid)
         DrawTrendLine(OBJ_PREFIX+"UMid", t1, chUY1+chUOff/2, t2, chUY2+chUOff/2, ChUpCol, 1, ChExtend, true);

      DrawLabel(OBJ_PREFIX+"ULbl", t2, chUY2,
                "UP (" + IntegerToString(chUPvts) + "P)", ChUpCol, 8);
     }

   if(chDOn && chDX1 >= 0)
     {
      t1 = iTime(Symbol(), Period(), totalBars - 1 - chDX1);
      t2 = iTime(Symbol(), Period(), totalBars - 1 - chDX2);

      DrawTrendLine(OBJ_PREFIX+"DBase", t1, chDY1, t2, chDY2, ChDnCol, 2, ChExtend);
      DrawTrendLine(OBJ_PREFIX+"DPar",  t1, chDY1+chDOff, t2, chDY2+chDOff, ChDnCol, 2, ChExtend);
      if(ChShowMid)
         DrawTrendLine(OBJ_PREFIX+"DMid", t1, chDY1+chDOff/2, t2, chDY2+chDOff/2, ChDnCol, 1, ChExtend, true);

      DrawLabel(OBJ_PREFIX+"DLbl", t2, chDY2,
                "DN (" + IntegerToString(chDPvts) + "P)", ChDnCol, 8);
     }
  }

//+------------------------------------------------------------------+
//| Calculate Breakout Levels & Counters                            |
//+------------------------------------------------------------------+
void CalculateBreakouts()
  {
   int currentBar = iBars(Symbol(), Period()) - 1;
   double cl[];
   ArraySetAsSeries(cl, true);
   CopyClose(Symbol(), Period(), 1, 1, cl);
   double closePrev = cl[0];

   // Up channel boundaries at current bar
   if(chUOn && chUX1 >= 0)
     {
      double base = LinePrice(chUX1, chUY1, chUX2, chUY2, currentBar);
      upChLower = base;
      upChUpper = base + chUOff;
     }
   else { upChUpper = 0; upChLower = 0; }

   // Down channel boundaries
   if(chDOn && chDX1 >= 0)
     {
      double base = LinePrice(chDX1, chDY1, chDX2, chDY2, currentBar);
      dnChUpper = base;
      dnChLower = base + chDOff;
     }
   else { dnChUpper = 0; dnChLower = 0; }

   // Consecutive counters
   upAboveCount = (upChUpper > 0 && closePrev > upChUpper) ? upAboveCount + 1 : 0;
   upBelowCount = (upChLower > 0 && closePrev < upChLower) ? upBelowCount + 1 : 0;
   dnAboveCount = (dnChUpper > 0 && closePrev > dnChUpper) ? dnAboveCount + 1 : 0;
   dnBelowCount = (dnChLower > 0 && closePrev < dnChLower) ? dnBelowCount + 1 : 0;
  }

//+------------------------------------------------------------------+
//| Generate Signals                                                 |
//+------------------------------------------------------------------+
void GenerateSignals()
  {
   int currentBar = iBars(Symbol(), Period()) - 1;

   sigBuy  = false; sigSell  = false;
   upBOBuy = false; upBOSell = false;
   dnBOBuy = false; dnBOSell = false;

   // ST Flip signals + EMA + MTF
   if(SigEnabled)
     {
      bool emaOK_buy  = EfEnabled ? emaAllAbove : true;
      bool emaOK_sell = EfEnabled ? emaAllBelow : true;

      if(stFlipBull && allowBuy  && emaOK_buy  && (currentBar - lastSTBuyBar)  >= SigCooldown)
         sigBuy  = true;
      if(stFlipBear && allowSell && emaOK_sell && (currentBar - lastSTSellBar) >= SigCooldown)
         sigSell = true;
     }

   // Up channel breakout
   if(BoUpEnabled)
     {
      bool upCoolOK = (currentBar - lastUpBOBar) >= BoUpCooldown;
      if(upCoolOK)
        {
         if(upAboveCount == BoUpConfirm) { upBOBuy  = true; lastUpBOBar = currentBar; }
         if(upBelowCount == BoUpConfirm) { upBOSell = true; lastUpBOBar = currentBar; }
        }
     }

   // Down channel breakout
   if(BoDnEnabled)
     {
      bool dnCoolOK = (currentBar - lastDnBOBar) >= BoDnCooldown;
      if(dnCoolOK)
        {
         if(dnAboveCount == BoDnConfirm) { dnBOBuy  = true; lastDnBOBar = currentBar; }
         if(dnBelowCount == BoDnConfirm) { dnBOSell = true; lastDnBOBar = currentBar; }
        }
     }

   // Combined — MTF must agree
   bool anyBuy  = (sigBuy  || upBOBuy  || dnBOBuy)  && allowBuy;
   bool anySell = (sigSell || upBOSell || dnBOSell) && allowSell;

   doLong  = anyBuy;
   doShort = anySell;

   if(sigBuy  || upBOBuy  || dnBOBuy)  { lastSTBuyBar  = currentBar; }
   if(sigSell || upBOSell || dnBOSell) { lastSTSellBar = currentBar; }
  }

//+------------------------------------------------------------------+
//| Position Sizing — Risk % of equity                              |
//+------------------------------------------------------------------+
double CalculateLotSize(double slPoints)
  {
   if(slPoints <= 0) return SymbolInfoDouble(Symbol(), SYMBOL_VOLUME_MIN);

   double equity     = AccountInfoDouble(ACCOUNT_EQUITY);
   double riskAmount = equity * RiskPercent / 100.0;
   double tickValue  = SymbolInfoDouble(Symbol(), SYMBOL_TRADE_TICK_VALUE);
   double tickSize   = SymbolInfoDouble(Symbol(), SYMBOL_TRADE_TICK_SIZE);
   double lotStep    = SymbolInfoDouble(Symbol(), SYMBOL_VOLUME_STEP);
   double minLot     = SymbolInfoDouble(Symbol(), SYMBOL_VOLUME_MIN);
   double maxLot     = SymbolInfoDouble(Symbol(), SYMBOL_VOLUME_MAX);

   if(tickSize <= 0 || tickValue <= 0) return minLot;

   double lotSize = riskAmount / (slPoints / tickSize * tickValue);
   lotSize = MathFloor(lotSize / lotStep) * lotStep;
   lotSize = MathMax(minLot, MathMin(maxLot, lotSize));

   return lotSize;
  }

//+------------------------------------------------------------------+
//| Execute Trades                                                   |
//+------------------------------------------------------------------+
void ExecuteTrades()
  {
   if(!doLong && !doShort) return;

   double ask   = SymbolInfoDouble(Symbol(), SYMBOL_ASK);
   double bid   = SymbolInfoDouble(Symbol(), SYMBOL_BID);
   double point = SymbolInfoDouble(Symbol(), SYMBOL_POINT);
   int    digits= (int)SymbolInfoInteger(Symbol(), SYMBOL_DIGITS);

   double slDist = atrVal * SlATRMult;
   double tpDist = slDist * SlRR;

   bool hasLong  = PositionSelect(Symbol()) && posInfo.Type() == POSITION_TYPE_BUY;
   bool hasShort = PositionSelect(Symbol()) && posInfo.Type() == POSITION_TYPE_SELL;

   // Close opposite if enabled
   if(CloseOnOpposite)
     {
      if(doLong  && hasShort) trade.PositionClose(Symbol());
      if(doShort && hasLong)  trade.PositionClose(Symbol());
     }

   // Open Long
   if(doLong && !hasLong)
     {
      double sl     = NormalizeDouble(ask - slDist, digits);
      double tp     = NormalizeDouble(ask + tpDist, digits);
      double slPts  = ask - sl;
      double lots   = CalculateLotSize(slPts);

      if(trade.Buy(lots, Symbol(), ask, sl, tp, "PC-BUY"))
        {
         activeSigDir = 1;
         Print("LONG opened: Lots=", lots, " SL=", sl, " TP=", tp);
         DrawSignalLabel(OBJ_PREFIX+"SigBuy"+IntegerToString(iBars(Symbol(),Period())),
                         iTime(Symbol(),Period(),1), ask, true);
        }
     }

   // Open Short
   if(doShort && !hasShort)
     {
      double sl     = NormalizeDouble(bid + slDist, digits);
      double tp     = NormalizeDouble(bid - tpDist, digits);
      double slPts  = sl - bid;
      double lots   = CalculateLotSize(slPts);

      if(trade.Sell(lots, Symbol(), bid, sl, tp, "PC-SELL"))
        {
         activeSigDir = -1;
         Print("SHORT opened: Lots=", lots, " SL=", sl, " TP=", tp);
         DrawSignalLabel(OBJ_PREFIX+"SigSell"+IntegerToString(iBars(Symbol(),Period())),
                         iTime(Symbol(),Period(),1), bid, false);
        }
     }

   // Update active direction from live position
   if(PositionSelect(Symbol()))
     {
      activeSigDir = (posInfo.Type() == POSITION_TYPE_BUY) ? 1 : -1;
     }
   else
      activeSigDir = 0;
  }

//+------------------------------------------------------------------+
//| DASHBOARD — Full MTF Table                                      |
//+------------------------------------------------------------------+
void DrawDashboard()
  {
   int    xBase, yBase;
   GetDashOrigin(xBase, yBase);

   int cellW  = 90;
   int cellH  = 18;
   int col0W  = 70;

   // Header
   DrawDashCell("HDR", xBase, yBase, col0W + cellW * 5, cellH,
                "MTF DIRECTIONAL BIAS", clrWhite, C'26,26,46', 9, true);

   // Column headers
   string colHdrs[6] = {"TF",
                        IntegerToString(DashEMA1)+" EMA",
                        IntegerToString(DashEMA2)+" EMA",
                        IntegerToString(DashEMA3)+" EMA",
                        "BIAS","SCORE"};
   int colWidths[6]  = {col0W, cellW, cellW, cellW, cellW+10, cellW-10};
   int xCur = xBase;
   for(int c = 0; c < 6; c++)
     {
      DrawDashCell("CH"+IntegerToString(c), xCur, yBase+cellH, colWidths[c], cellH,
                   colHdrs[c], clrWhite, C'15,52,96', 8, true);
      xCur += colWidths[c];
     }

   // TF rows
   string tfNames[6] = {"5m","15m","30m","1H","4H","Daily"};
   int    scores[6]  = {s5, s15, s30, s60, s240, sD};
   bool   a1s[6]     = {a5_1, a15_1, a30_1, a60_1, a240_1, aD_1};
   bool   a2s[6]     = {a5_2, a15_2, a30_2, a60_2, a240_2, aD_2};
   bool   a3s[6]     = {a5_3, a15_3, a30_3, a60_3, a240_3, aD_3};

   for(int r = 0; r < 6; r++)
     {
      int    yRow = yBase + cellH * (r + 2);
      color  rBg  = (r % 2 == 0) ? C'22,33,62' : C'26,26,46';
      int    sc   = scores[r];

      xCur = xBase;
      // TF name
      DrawDashCell("R"+IntegerToString(r)+"C0", xCur, yRow, colWidths[0], cellH,
                   tfNames[r], clrWhite, rBg, 8, false);
      xCur += colWidths[0];
      // EMA1 dot
      DrawDashCell("R"+IntegerToString(r)+"C1", xCur, yRow, colWidths[1], cellH,
                   (a1s[r] ? "+" : "-"), a1s[r]?clrLime:clrRed, rBg, 10, true);
      xCur += colWidths[1];
      // EMA2 dot
      DrawDashCell("R"+IntegerToString(r)+"C2", xCur, yRow, colWidths[2], cellH,
                   (a2s[r] ? "+" : "-"), a2s[r]?clrLime:clrRed, rBg, 10, true);
      xCur += colWidths[2];
      // EMA3 dot
      DrawDashCell("R"+IntegerToString(r)+"C3", xCur, yRow, colWidths[3], cellH,
                   (a3s[r] ? "+" : "-"), a3s[r]?clrLime:clrRed, rBg, 10, true);
      xCur += colWidths[3];
      // Bias text
      DrawDashCell("R"+IntegerToString(r)+"C4", xCur, yRow, colWidths[4], cellH,
                   BiasText(sc), clrWhite, BiasColor(sc), 8, true);
      xCur += colWidths[4];
      // Score
      DrawDashCell("R"+IntegerToString(r)+"C5", xCur, yRow, colWidths[5], cellH,
                   IntegerToString(sc)+"/3", BiasColor(sc), rBg, 8, true);
     }

   // Overall row
   int    totalSc = (s5+s15+s30+s60+s240+sD) / 6;
   int    yOver   = yBase + cellH * 8;
   xCur = xBase;
   DrawDashCell("OVR0", xCur, yOver, colWidths[0], cellH, "OVERALL", clrYellow, C'15,52,96', 8, true);
   xCur += colWidths[0];
   DrawDashCell("OVR1", xCur, yOver, colWidths[1]+colWidths[2]+colWidths[3], cellH, "", clrWhite, C'15,52,96', 8, false);
   xCur += colWidths[1]+colWidths[2]+colWidths[3];
   DrawDashCell("OVR4", xCur, yOver, colWidths[4], cellH, BiasText(totalSc), clrWhite, BiasColor(totalSc), 8, true);
   xCur += colWidths[4];
   DrawDashCell("OVR5", xCur, yOver, colWidths[5], cellH,
                IntegerToString(mtfBullCount)+"/6 Bull", clrYellow, C'15,52,96', 8, true);

   // Trend lock row
   string tLock  = allowBuy  ? "BUY SIGNALS ONLY" : allowSell ? "SELL SIGNALS ONLY" : "NO SIGNALS (WAIT)";
   color  tLockC = allowBuy  ? C'0,230,118' : allowSell ? C'255,82,82' : C'255,214,0';
   color  tLockB = allowBuy  ? C'0,60,30'   : allowSell ? C'60,0,0'    : C'60,50,0';
   int    yTrend = yBase + cellH * 9;
   xCur = xBase;
   DrawDashCell("TR0", xCur, yTrend, colWidths[0], cellH, "TREND", C'224,224,224', C'27,27,58', 8, false);
   xCur += colWidths[0];
   DrawDashCell("TR1", xCur, yTrend, colWidths[1]+colWidths[2]+colWidths[3]+colWidths[4]+colWidths[5], cellH,
                tLock, clrWhite, tLockB, 8, true);

   // Position row
   string posText = (activeSigDir == 1) ? "LONG" : (activeSigDir == -1) ? "SHORT" : "FLAT";
   color  posBg   = (activeSigDir == 1) ? C'0,60,30' : (activeSigDir == -1) ? C'60,0,0' : C'50,50,50';
   string posLine = "Pos: " + posText +
                    "  |  MTF: " + IntegerToString(AutoMinTFs) + "TF" +
                    "  |  EMA: " + (EfEnabled ? "ON" : "OFF") +
                    "  |  ST: "  + (SigEnabled ? "ON" : "OFF") +
                    "  |  BO: "  + (BoUpEnabled || BoDnEnabled ? "ON" : "OFF") +
                    "  |  Risk: " + DoubleToString(RiskPercent, 1) + "%";
   int    yPos    = yBase + cellH * 10;
   xCur = xBase;
   DrawDashCell("PS0", xCur, yPos, colWidths[0], cellH, "POS", C'224,224,224', C'27,27,58', 8, false);
   xCur += colWidths[0];
   DrawDashCell("PS1", xCur, yPos, colWidths[1]+colWidths[2]+colWidths[3]+colWidths[4]+colWidths[5], cellH,
                posLine, clrWhite, posBg, 8, true);

   ChartRedraw();
  }

//+------------------------------------------------------------------+
//| Get Dashboard origin coordinates                                 |
//+------------------------------------------------------------------+
void GetDashOrigin(int &x, int &y)
  {
   int chartW = (int)ChartGetInteger(0, CHART_WIDTH_IN_PIXELS);
   int chartH = (int)ChartGetInteger(0, CHART_HEIGHT_IN_PIXELS);
   int dashW  = 530;
   int dashH  = 200;
   switch(DashPosition)
     {
      case DASH_TOP_LEFT:     x = 10;            y = 30;            break;
      case DASH_BOTTOM_RIGHT: x = chartW-dashW-10; y = chartH-dashH-30; break;
      case DASH_BOTTOM_LEFT:  x = 10;            y = chartH-dashH-30; break;
      default:                x = chartW-dashW-10; y = 30;          break;
     }
  }

//+------------------------------------------------------------------+
//| Draw a dashboard cell (rectangle + label)                       |
//+------------------------------------------------------------------+
void DrawDashCell(string id, int x, int y, int w, int h,
                  string text, color textCol, color bgCol,
                  int fontSize, bool centered)
  {
   string rectName  = OBJ_PREFIX + "R_" + id;
   string labelName = OBJ_PREFIX + "L_" + id;

   if(ObjectFind(0, rectName) < 0)
      ObjectCreate(0, rectName, OBJ_RECTANGLE_LABEL, 0, 0, 0);
   ObjectSetInteger(0, rectName, OBJPROP_XDISTANCE,  x);
   ObjectSetInteger(0, rectName, OBJPROP_YDISTANCE,  y);
   ObjectSetInteger(0, rectName, OBJPROP_XSIZE,      w);
   ObjectSetInteger(0, rectName, OBJPROP_YSIZE,      h);
   ObjectSetInteger(0, rectName, OBJPROP_BGCOLOR,    bgCol);
   ObjectSetInteger(0, rectName, OBJPROP_BORDER_TYPE,BORDER_FLAT);
   ObjectSetInteger(0, rectName, OBJPROP_COLOR,      C'50,50,70');
   ObjectSetInteger(0, rectName, OBJPROP_CORNER,     CORNER_LEFT_UPPER);
   ObjectSetInteger(0, rectName, OBJPROP_BACK,       false);
   ObjectSetInteger(0, rectName, OBJPROP_SELECTABLE, false);

   if(ObjectFind(0, labelName) < 0)
      ObjectCreate(0, labelName, OBJ_LABEL, 0, 0, 0);
   int lx = centered ? x + w/2 : x + 4;
   ObjectSetInteger(0, labelName, OBJPROP_XDISTANCE,  lx);
   ObjectSetInteger(0, labelName, OBJPROP_YDISTANCE,  y + h/2 - fontSize/2);
   ObjectSetInteger(0, labelName, OBJPROP_CORNER,     CORNER_LEFT_UPPER);
   ObjectSetInteger(0, labelName, OBJPROP_ANCHOR,     centered ? ANCHOR_CENTER : ANCHOR_LEFT);
   ObjectSetInteger(0, labelName, OBJPROP_COLOR,      textCol);
   ObjectSetInteger(0, labelName, OBJPROP_FONTSIZE,   fontSize);
   ObjectSetString(0,  labelName, OBJPROP_FONT,       "Arial Bold");
   ObjectSetString(0,  labelName, OBJPROP_TEXT,       text);
   ObjectSetInteger(0, labelName, OBJPROP_BACK,       false);
   ObjectSetInteger(0, labelName, OBJPROP_SELECTABLE, false);
  }

//+------------------------------------------------------------------+
//| Bias text helper                                                 |
//+------------------------------------------------------------------+
string BiasText(int sc)
  {
   if(sc == 3)  return "STRONG BULL";
   if(sc == -3) return "STRONG BEAR";
   if(sc >= 1)  return "BULLISH";
   if(sc <= -1) return "BEARISH";
   return "NEUTRAL";
  }

//+------------------------------------------------------------------+
//| Bias color helper                                                |
//+------------------------------------------------------------------+
color BiasColor(int sc)
  {
   if(sc == 3)  return C'0,230,118';
   if(sc == -3) return C'255,23,68';
   if(sc >= 1)  return C'0,150,80';
   if(sc <= -1) return C'180,0,40';
   return C'100,100,100';
  }

//+------------------------------------------------------------------+
//| Draw Trend Line on Chart                                        |
//+------------------------------------------------------------------+
void DrawTrendLine(string name, datetime t1, double p1, datetime t2, double p2,
                   color clr, int width, bool extendRight, bool dashed=false)
  {
   if(ObjectFind(0, name) < 0)
      ObjectCreate(0, name, OBJ_TREND, 0, t1, p1, t2, p2);
   ObjectSetInteger(0, name, OBJPROP_TIME,   0, t1);
   ObjectSetDouble(0,  name, OBJPROP_PRICE,  0, p1);
   ObjectSetInteger(0, name, OBJPROP_TIME,   1, t2);
   ObjectSetDouble(0,  name, OBJPROP_PRICE,  1, p2);
   ObjectSetInteger(0, name, OBJPROP_COLOR,  clr);
   ObjectSetInteger(0, name, OBJPROP_WIDTH,  width);
   ObjectSetInteger(0, name, OBJPROP_RAY_RIGHT, extendRight);
   ObjectSetInteger(0, name, OBJPROP_STYLE,  dashed ? STYLE_DASH : STYLE_SOLID);
   ObjectSetInteger(0, name, OBJPROP_SELECTABLE, false);
   ObjectSetInteger(0, name, OBJPROP_BACK,   true);
  }

//+------------------------------------------------------------------+
//| Draw text label on chart                                        |
//+------------------------------------------------------------------+
void DrawLabel(string name, datetime t, double price, string text, color clr, int fontSize)
  {
   if(ObjectFind(0, name) < 0)
      ObjectCreate(0, name, OBJ_TEXT, 0, t, price);
   ObjectSetInteger(0, name, OBJPROP_TIME,    t);
   ObjectSetDouble(0,  name, OBJPROP_PRICE,   price);
   ObjectSetString(0,  name, OBJPROP_TEXT,    text);
   ObjectSetInteger(0, name, OBJPROP_COLOR,   clr);
   ObjectSetInteger(0, name, OBJPROP_FONTSIZE,fontSize);
   ObjectSetString(0,  name, OBJPROP_FONT,    "Arial Bold");
   ObjectSetInteger(0, name, OBJPROP_SELECTABLE, false);
  }

//+------------------------------------------------------------------+
//| Draw horizontal line                                            |
//+------------------------------------------------------------------+
void DrawHLine(string name, double price, color clr, int width, ENUM_LINE_STYLE style)
  {
   if(ObjectFind(0, name) < 0)
      ObjectCreate(0, name, OBJ_HLINE, 0, 0, price);
   ObjectSetDouble(0,  name, OBJPROP_PRICE, price);
   ObjectSetInteger(0, name, OBJPROP_COLOR, clr);
   ObjectSetInteger(0, name, OBJPROP_WIDTH, width);
   ObjectSetInteger(0, name, OBJPROP_STYLE, style);
   ObjectSetInteger(0, name, OBJPROP_SELECTABLE, false);
  }

//+------------------------------------------------------------------+
//| Draw Buy/Sell signal arrow label                                |
//+------------------------------------------------------------------+
void DrawSignalLabel(string name, datetime t, double price, bool isBuy)
  {
   double arrowPrice = isBuy ? price - atrVal * 0.5 : price + atrVal * 0.5;
   if(ObjectFind(0, name) < 0)
      ObjectCreate(0, name, OBJ_ARROW, 0, t, arrowPrice);
   ObjectSetInteger(0, name, OBJPROP_TIME,       0, t);
   ObjectSetDouble(0,  name, OBJPROP_PRICE,      0, arrowPrice);
   ObjectSetInteger(0, name, OBJPROP_ARROWCODE,  (long)(isBuy ? 233 : 234));
   ObjectSetInteger(0, name, OBJPROP_COLOR,      isBuy ? clrLime : clrRed);
   ObjectSetInteger(0, name, OBJPROP_WIDTH,      2);
   ObjectSetInteger(0, name, OBJPROP_SELECTABLE, false);
   ObjectSetInteger(0, name, OBJPROP_BACK,       false);
  }

//+------------------------------------------------------------------+
//| Delete all dashboard objects                                    |
//+------------------------------------------------------------------+
void DeleteDashboard()
  {
   int total = ObjectsTotal(0);
   for(int i = total - 1; i >= 0; i--)
     {
      string name = ObjectName(0, i);
      if(StringFind(name, OBJ_PREFIX) == 0)
         ObjectDelete(0, name);
     }
  }

//+------------------------------------------------------------------+
//| Delete channel drawing objects                                  |
//+------------------------------------------------------------------+
void DeleteChannelObjects()
  {
   string names[] = {"UBase","UPar","UMid","ULbl","DBase","DPar","DMid","DLbl"};
   for(int i = 0; i < ArraySize(names); i++)
      ObjectDelete(0, OBJ_PREFIX + names[i]);
  }

//+------------------------------------------------------------------+
//| END — Parallel Channel EA v1.0                                  |
//+------------------------------------------------------------------+

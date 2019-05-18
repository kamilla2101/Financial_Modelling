####Created Mortgage Class with a constructor, defining psa, yeras of mortgage pool, Payment frequenct(CY),APR(r),and Total Beiginnging Balance (P)
class Mortgage:

    def __init__(self, P=0, yrs=30, CY=12, r=0.075, psa=1):
        self.psa = psa
        self.yrs = yrs
        self.CY = CY
        self.r = r
        self.n = yrs * CY
        self.pr = r / CY
        self.P = [P]
        self.int = [0]
        self.pmt = [0]
        self.prn = [0]
        self.pre = [0]
        self.totprn = [0]
        self.totcf =[0]
        if not isinstance(self, defaultCir):
            self.generate()
            self.generate_report()

    def calprn(self, BB, t):
        return BB * self.pr / (1 - (1 + self.pr) ** (self.n-t+1))

    def calint(self, BB):
        return BB * self.pr

    def calpre(self, t, BB):
        if t <= 30:
            cpr = t * 0.002 * self.psa
        else:
            cpr = 0.06 * self.psa
        return (1 - (1 - cpr)**(1/12))*(BB + self.calprn(BB, t))

    def generate(self):
        for t in range(1, self.n + 1):
            self.P.append(self.P[t-1] - self.prn[t-1] - self.pre[t-1])
            self.prn.append(- self.calprn(self.P[t], t))
            self.int.append(self.calint(self.P[t]))
            self.pmt.append(- self.calprn(self.P[t], t) + self.int[t])
            self.pre.append(self.calpre(t, self.P[t]))
            self.totprn.append(self.prn[t] + self.pre[t])
            self.totcf.append(self.int[t] + self.totprn[t])

    def getn(self):
        return self.n

    def gettotcf(self):
        return self.totcf

    def gettotprn(self):
        return self.totprn

    def getpre(self):
        return self.pre

    def getprn(self):
        return self.prn

    def getpmt(self):
        return self.pmt

    def getp(self):
        return self.P

    def getint(self):
        return self.int

    def getcy(self):
        return self.CY

    def getsmm(self, t):
        if t <= 30:
            cpr = t * 0.002 * self.psa
        else:
            cpr = 0.06 * self.psa
        return 1 - (1 - cpr) ** (1 / 12)

#####Wrote calculated data onto csv file
    def generate_report(self):
        with open('MortgageAnalysis.csv', 'w') as csvfile:
            csvfile.write(
                format("month", "<15s") + "," + format("years", "<15s") + "," + format("Balance", "<15s") + "," + format("Payment",
                                                                                                       "<15s") + "," + format(
                    "interest", "<15s") + "," + format("Principal", "<15s") + "," + format("Prepayment", "<15s") + "," + format(
                    "Total Prn", "<15s") + "," + format("Total Cash", "<15s"))
            csvfile.write("\n")
            for i in range(self.n + 1):
                line = [i, i / self.CY, self.P[i], self.pmt[i], self.int[i], self.prn[i],
                        self.pre[i], self.totprn[i], self.totcf[i]]

                for j in range(len(line)):
                    csvfile.write(format(str(round(line[j], 2)), "<15s"))
                    if j < 8:
                        csvfile.write(',')
                csvfile.write("\n")

            csvfile.close()


class defaultCir(Mortgage):

    def __init__(self, P=0,  yrs=30, CY=12, r=0.075, psa=1, Rr=0.6, ADR=0.02):
        super().__init__()
        self.P = [P]
        self.psa = psa
        self.yrs = yrs
        self.CY = CY
        self.r = r
        self.__Rr = Rr
        self.__ADR = ADR
        self.__default = [0]
        self.__recovery = [0]
        self.__intmp = [0]
        self.__rcvintdef = [0]
        self.__rcvprndef = [0]
        self.__totlossprn = [0]
        self.default_generate()
        self.generate_report()

    def caldefault(self, BB, Prn, Pre):
        return (1 - (1 - self.__ADR)**(1/12))*(BB - Prn - Pre)

    def calrecv(self, default):
        return self.__Rr * default

    def calrcvint(self, default):
        return self.pr * default

    def default_generate(self):
        for t in range(1, self.n + 1):
            self.P.append(self.P[t-1] - self.prn[t-1] - self.pre[t-1]- self.__default[t-1])
            self.prn.append(- self.calprn(self.P[t], t))
            self.pre.append(self.calpre(t, self.P[t]))
            self.__default.append(self.caldefault(self.P[t], self.prn[t], self.pre[t]))
            self.__recovery.append(self.calrecv(self.__default[t]))
            self.__intmp.append((self.P[t] - self.__default[t]) * self.pr)
            self.__rcvintdef.append(self.__default[t] * self.pr)
            self.__rcvprndef.append(self.__recovery[t] - self.__rcvintdef[t])
            self.totprn.append(self.prn[t] + self.pre[t]+self.__rcvprndef[t])
            self.int.append(self.__intmp[t] + self.__rcvintdef[t])
            self.__totlossprn.append(self.__rcvprndef[t] - self.__default[t])

    def generate_report(self):
        with open('MortgageAnalysis.csv', 'w') as csvfile:
            csvfile.write(
                format("month", "<15s") + "," + format("years", "<15s") + "," + format("Balance", "<15s") + "," + format("Principal",
                                                                                                       "<15s") + "," + format(
                    "Prepayment", "<15s") + "," + format("Default", "<15s") + "," + format("Recovery", "<15s") + "," + format(
                    "Intmp", "<15s") + "," + format("RcvIntDef", "<15s") + "," + format("RcvPrnDef","<15s") + "," + format("TotPrn","<15s") + "," + format("TotInt","<15s") + "," + format("Totloss","<15s"))
            csvfile.write("\n")
            for i in range(self.n + 1):
                line = [i, i / self.CY, self.P[i], self.prn[i], self.pre[i], self.__default[i],
                        self.__recovery[i], self.__intmp[i], self.__rcvintdef[i], self.__rcvprndef[i], self.totprn[i], self.int[i], self.__totlossprn[i]]

                for j in range(len(line)):
                    csvfile.write(format(str(round(line[j], 2)), "<15s"))
                    if j < 12:
                        csvfile.write(',')
                csvfile.write("\n")

            csvfile.close()

    def getdefault(self):
        return self.__default

    def getrecovery(self):
        return self.__recovery

    def getintmp(self):
        return self.__intmp

    def getrcvint(self):
        return self.__rcvintdef

    def getrcvprn(self):
        return self.__rcvprndef

    def totloss(self):
        return self.__totlossprn
####Input you values
def main():
    a,b,c,d,e = eval(input())
    
    MP = Mortgage(a,b,c,d,e)
    
main()

#### Graphical Representation of Prepayment 
import pandas as pd
from datetime import date
import numpy as np
from collections import OrderedDict
from dateutil.relativedelta import *
import matplotlib.pyplot as plt
from IPython.core.pylabtools import figsize

df = pd.read_csv("MortgageAnalysis100.csv")
df

df1 = pd.read_csv("MortgageAnalysis200.csv")
df1



plt.title("PSA Behaviour")
plt.plot(df, label = 'Scenario 100')
plt.plot(df1, label = 'Scenario 200')
plt.legend()
plt.show()



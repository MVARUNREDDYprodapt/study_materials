from abc import ABC, abstractmethod
from enum import Enum, auto
# OTT ENUMS:
class OTT(Enum):
    HOTSTAR = auto()
    PRIME = auto()
    NETFLIX = auto()
    SPOTIFY = auto()

# data/voice/sms usage template
class Usage():
    '''
    represents the customers 30 usage requirements
    '''
    def __init__(self, voiceMinutes, smsCount, dataMB, ottRequirements):
        self.voiceMins = voiceMinutes
        self.smsCount = smsCount
        self.dataMB = dataMB
        self.ottRequirements = ottRequirements

# template for plan quote
class PlanQuote():
    '''
    represents the cost breakdown for any plan
    '''
    def __init__(self, planName, rentalFor30Days, dataOverageCost, smsOverageCost, voiceOverageCost, total, OTTsOffered):
        self.planName = planName
        self.rentalFor30Days = rentalFor30Days
        self.dataOverageCost = dataOverageCost
        self.smsOverageCost = smsOverageCost
        self.voiceOverageCost = voiceOverageCost
        self.OTTsOffered = OTTsOffered
        self.total = total
    
    def __str__(self):
        '''
        for formatting the output, print(obj), we'll get a formatted output as below instead of <obj at 0xsmth>
        '''
        otts = ", ".join([ott.name for ott in self.OTTsOffered]) if self.OTTsOffered else "None"
        
        return (f"Plan: {self.planName}\n"
                f"Rental for 30 Days: ₹{self.rentalFor30Days:.2f}\n"
                f"Data Overage cost: ₹{self.dataOverageCost:.2f}\n"
                f"Voice Overage cost: ₹{self.voiceOverageCost:.2f}\n"
                f"SMS Overage cost: ₹{self.smsOverageCost:.2f}\n"
                f"OTTs Offered: {otts}\n"
                f"TOTAL COST: ₹{self.total:.2f}\n")


# main abstract class for all plans
class Plan(ABC):
    '''
    main abstract class for all plans
    '''
    def __init__(self, name, cost, validityDays, ottIncluded):
        self.name = name
        self.cost = cost
        self.validityDays = validityDays
        self.ottIncluded = ottIncluded

    def normalizedRental(self): # returns the 30 day normalized cost for any plan
        return (self.cost / self.validityDays) * 30 

    @abstractmethod
    def calculateCost(self, usage: Usage) -> PlanQuote:
        pass

'''
INDIVIDUAL PLAN DEFINITION
'''

class BasicLite(Plan):
    '''
    Basic Lite — ₹249, 28 days
        o Data: 1 GB/day (overage: ₹0.70/10 MB)
        o Voice: 100 mins included, then ₹0.75/min
        o SMS: ₹0.20/SMS
        o OTT: none
    '''
    def __init__(self):
        super().__init__("Basic Lite", 249, 28, {})
        self.dataPerDayGB = 1
        self.voiceQuota = 100
        self.smsQuota = 0

    def calculateCost(self, usage: Usage):
        rental = self.normalizedRental()

        # data totals
        dataQuotaMB = self.dataPerDayGB * 30 * 1024
        dataOver = max(0, usage.dataMB - dataQuotaMB)
        dataOverageCost = (dataOver / 10) * 0.70

        # sms totals
        smsOver = max(0, usage.smsCount - self.smsQuota)
        smsOverageCost = smsOver * 0.20

        # voice totals
        voiceOver = max(0, usage.voiceMins - self.voiceQuota)
        voiceOverageCost = voiceOver * 0.75

        total = rental + dataOverageCost + smsOverageCost + voiceOverageCost
        
        return PlanQuote(self.name, rental, dataOverageCost, smsOverageCost, voiceOverageCost, total, self.ottIncluded)
    
class Saver30(Plan):
    '''
    Saver 30 — ₹499, 30 days
        o Data: 1.5 GB/day (overage: ₹0.70/10 MB)
        o Voice: 300 mins included, then ₹0.75/min
        o SMS: 100 free, then ₹0.2/SMS
        o OTT: Hotstar
    '''
    def __init__(self):
        super().__init__("Saver 30", 499, 30, {OTT.HOTSTAR})
        self.dataPerDayGB = 1.5
        self.voiceQuota = 300
        self.smsQuota = 100

    def calculateCost(self, usage: Usage):
        rental = self.normalizedRental()

        # data totals
        dataQuotaMB = self.dataPerDayGB * 30 * 1024
        dataOver = max(0, usage.dataMB - dataQuotaMB)
        dataOverageCost = (dataOver / 10) * 0.70

        # sms totals
        smsOver = max(0, usage.smsCount - self.smsQuota)
        smsOverageCost = smsOver * 0.20

        # voice totals
        voiceOver = max(0, usage.voiceMins - self.voiceQuota)
        voiceOverageCost = voiceOver * 0.75

        total = rental + dataOverageCost + smsOverageCost + voiceOverageCost
        
        return PlanQuote(self.name, rental, dataOverageCost, smsOverageCost, voiceOverageCost, total, self.ottIncluded)
        
class UnlimitedTalk30(Plan):
    '''
     Unlimited Talk 30 — ₹650, 30 days
        o Data: 5 GB total included (overage: ₹0.70/10 MB)
        o Voice: Unlimited
        o SMS: Unlimited
        o OTT: Spotify
    '''
    def __init__(self):
        super().__init__("Unlimited Talk 30", 650, 30, {OTT.SPOTIFY})
        self.dataQuotaMB = 5 * 1024

    def calculateCost(self, usage: Usage):
        rental = self.normalizedRental()

        # data totals
        dataOver = max(0, usage.dataMB - self.dataQuotaMB)
        dataOverageCost = (dataOver / 10) * 0.70

        # sms totals
        smsOverageCost = 0

        # voice totals
        voiceOverageCost = 0

        total = rental + dataOverageCost + smsOverageCost + voiceOverageCost
        
        return PlanQuote(self.name, rental, dataOverageCost, smsOverageCost, voiceOverageCost, total, self.ottIncluded)
        
class DataMax20(Plan):
    '''
    Data Max 20 — ₹749, 20 days
        o Data: Unlimited
        o Voice: 100 mins included, then ₹0.75/min
        o SMS: Unlimited
        o OTT: Hotstar
    '''
    def __init__(self):
        super().__init__("Saver 30", 749, 20, {OTT.HOTSTAR})
        self.voiceQuota = 100

    def calculateCost(self, usage: Usage):
        rental = self.normalizedRental()

        # data totals
        dataOverageCost = 0

        # sms totals
        smsOverageCost = 0

        # voice totals
        voiceOver = max(0, usage.voiceMins - self.voiceQuota)
        voiceOverageCost = voiceOver * 0.75

        total = rental + dataOverageCost + smsOverageCost + voiceOverageCost
        
        return PlanQuote(self.name, rental, dataOverageCost, smsOverageCost, voiceOverageCost, total, self.ottIncluded)

class StudentStream56(Plan):
    '''
    Student Stream 56 — ₹435, 30 days
        o Data: 2 GB/day (overage: ₹0.70/10 MB)
        o Voice: 300 mins included, then ₹0.75/min
        o SMS: 200 free, then ₹0.20/SMS
        o OTT: Spotify
    '''
    def __init__(self):
        super().__init__("Student Stream 56", 435, 30, {OTT.SPOTIFY})
        self.dataPerDayGB = 2
        self.voiceQuota = 300
        self.smsQuota = 200

    def calculateCost(self, usage: Usage):
        rental = self.normalizedRental()

        # data totals
        dataQuotaMB = self.dataPerDayGB * 30 * 1024
        dataOver = max(0, usage.dataMB - dataQuotaMB)
        dataOverageCost = (dataOver / 10) * 0.70

        # sms totals
        smsOver = max(0, usage.smsCount - self.smsQuota)
        smsOverageCost = smsOver * 0.20

        # voice totals
        voiceOver = max(0, usage.voiceMins - self.voiceQuota)
        voiceOverageCost = voiceOver * 0.75

        total = rental + dataOverageCost + smsOverageCost + voiceOverageCost
        
        return PlanQuote(self.name, rental, dataOverageCost, smsOverageCost, voiceOverageCost, total, self.ottIncluded)

class FamilyShare30(Plan):
    '''
    Family Share 30 — ₹500, 28 days
        o Data: 50 GB total for validity (overage: ₹0.70/10 MB)
        o Voice: 1000 mins included, then ₹0.6/min
        o SMS: 500 free, then ₹0.20/SMS
        o OTT: Amazon Prime
    '''
    def __init__(self):
        super().__init__("SaFamily Share 30", 500, 28, {OTT.PRIME})
        self.dataQuotaMB = 50 * 1024
        self.voiceQuota = 1000
        self.smsQuota = 500

    def calculateCost(self, usage: Usage):
        rental = self.normalizedRental()

        # data totals
        dataOver = max(0, usage.dataMB - self.dataQuotaMB)
        dataOverageCost = (dataOver / 10) * 0.70

        # sms totals
        smsOver = max(0, usage.smsCount - self.smsQuota)
        smsOverageCost = smsOver * 0.20

        # voice totals
        voiceOver = max(0, usage.voiceMins - self.voiceQuota)
        voiceOverageCost = voiceOver * 0.75

        total = rental + dataOverageCost + smsOverageCost + voiceOverageCost
        
        return PlanQuote(self.name, rental, dataOverageCost, smsOverageCost, voiceOverageCost, total, self.ottIncluded)

class DataMaxPlus30(Plan):
    '''
    Data Max Plus 30 — ₹1499, 30 days
        o Data: Unlimited
        o Voice: 300 mins included, then ₹0.75/min
        o SMS: 200 free, then ₹0.20/SMS
        o OTT: Amazon Prime + Hotstar
    '''
    def __init__(self):
        super().__init__("Data Max Plus 30", 1499, 30, {OTT.HOTSTAR, OTT.PRIME})
        self.voiceQuota = 300
        self.smsQuota = 200

    def calculateCost(self, usage: Usage):
        rental = self.normalizedRental()

        # data totals
        dataOverageCost = 0

        # sms totals
        smsOver = max(0, usage.smsCount - self.smsQuota)
        smsOverageCost = smsOver * 0.20

        # voice totals
        voiceOver = max(0, usage.voiceMins - self.voiceQuota)
        voiceOverageCost = voiceOver * 0.75

        total = rental + dataOverageCost + smsOverageCost + voiceOverageCost
        
        return PlanQuote(self.name, rental, dataOverageCost, smsOverageCost, voiceOverageCost, total, self.ottIncluded)

class PremiumUltra30(Plan):
    '''
     Premium Ultra 30 — ₹2999, 30 days
        o Data/Voice/SMS: Unlimited
        o OTT: Netflix + Amazon Prime + Hotstar + Spotify
    '''
    def __init__(self):
        super().__init__("Premium Ultra 30", 2999, 30, {OTT.HOTSTAR, OTT.PRIME, OTT.NETFLIX, OTT.SPOTIFY})

    def calculateCost(self, usage: Usage):
        rental = self.normalizedRental()

        # data totals
        dataOverageCost = 0

        # sms totals0
        smsOverageCost = 0

        # voice totals0
        voiceOverageCost = 0

        total = rental + dataOverageCost + smsOverageCost + voiceOverageCost
        
        return PlanQuote(self.name, rental, dataOverageCost, smsOverageCost, voiceOverageCost, total, self.ottIncluded)
    

'''
Optimiser
'''

class planOptimizer():
    def __init__(self, plans):
        self.plans = plans
    
    def recommend(self, usage: Usage):
        quotes = []
        for plan in self.plans:
            if not usage.ottRequirements.issubset(plan.ottIncluded):
                continue
            quote = plan.calculateCost(usage)
            quotes.append(quote)

            if not quotes:
                return None, []
            
            bestQuote = min(quotes, key=lambda q: q.total)
            return bestQuote, quotes

'''
test run
'''

if __name__ == "__main__":

    usage1 = Usage(voiceMinutes=200, smsCount=20, dataMB=2000, ottRequirements={OTT.HOTSTAR, OTT.NETFLIX})
    usage2 = Usage(voiceMinutes=1200, smsCount=20, dataMB=2000, ottRequirements={OTT.PRIME, OTT.HOTSTAR})
    usage3 = Usage(voiceMinutes=200, smsCount=210, dataMB=12000, ottRequirements={OTT.HOTSTAR})
    usage4 = Usage(voiceMinutes=200, smsCount=20, dataMB=2000, ottRequirements={OTT.SPOTIFY, OTT.NETFLIX})

    usage5 = Usage(voiceMinutes=650, smsCount=300, dataMB=8*1024, ottRequirements={})

    plans = [BasicLite(), Saver30(), PremiumUltra30(), DataMaxPlus30(), FamilyShare30(), StudentStream56(), DataMax20(), UnlimitedTalk30()]

    optimizer = planOptimizer(plans)
    bestPlan, allQuotes = optimizer.recommend(usage5)

    for q in allQuotes:
        print(q)

    if bestPlan:
        print(f"✅ Recommended Plan → {bestPlan.planName} (cheapest + meets OTT)")
    else:
        print("❌ No plan meets OTT requirements")



$ python -u "c:\Users\varunreddy.m\Desktop\training\python\test.py"
Traceback (most recent call last):
  File "c:\Users\varunreddy.m\Desktop\training\python\test.py", line 360, in <module>
    bestPlan, allQuotes = optimizer.recommend(usage5)
                          ~~~~~~~~~~~~~~~~~~~^^^^^^^^
  File "c:\Users\varunreddy.m\Desktop\training\python\test.py", line 333, in recommend
    if not usage.ottRequirements.issubset(plan.ottIncluded):
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AttributeError: 'dict' object has no attribute 'issubset'
(imsick) 

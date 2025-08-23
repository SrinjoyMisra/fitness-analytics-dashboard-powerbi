**//------------------- // Fitness Dashboard DAX Measures // Copy \& Paste in your Power BI Model //----------------------------- //**



// Gym Members count 

Users\_Count = DISTINCTCOUNT(Users\[UserID])



// Total Revenue

Revenue = SUM(Payments\[Amount])



// Total Expenses

Expanses = SUM(Expenses\[Amount])



// Profit Calculation

Profit = \[Revenue] - \[Expanses]



// Maximum Users in Calendar

Max\_users = MAXX(ALL('Calendar'\[Month],'Calendar'\[MonthIndex]), \[Users\_Count])



// Minimum Users in Calendar

Min\_users = MINX(ALL('Calendar'\[Month],'Calendar'\[MonthIndex]), \[Users\_Count])



// Conditional User Color based on Min/Max

Con\_Users\_Color =

VAR MaxUser = \[Max\_users]

VAR MinUser = \[Min\_users]

VAR Con\_Color = SELECTEDVALUE(ColorCodes\[Codes])

VAR Value\_Switch = SELECTEDVALUE(Min\_Max\_Switch\[Type])

VAR Min\_Color = IF(MinUser = \[Users\_Count], Con\_Color, "gray")

VAR Max\_Color = IF(MaxUser = \[Users\_Count], Con\_Color, "gray")

RETURN IF(Value\_Switch = "Max", Max\_Color, Min\_Color)



// Completed Membership Days

Complete\_Days =

DATEDIFF(

&nbsp;   SELECTEDVALUE(Users\[MembershipStart]),

&nbsp;   MIN(TODAY(), SELECTEDVALUE(Users\[MembershipEnd])),

&nbsp;   DAY

)



// Remaining Membership Days

RemainingDays =

DATEDIFF(

&nbsp;   SELECTEDVALUE(Users\[MembershipStart], TODAY()),

&nbsp;   SELECTEDVALUE(Users\[MembershipEnd]),

&nbsp;   DAY

)



// Left Days = Remaining - Completed

Left\_Days\_Count = \[RemainingDays] - \[Complete\_Days]



// Total Membership Days

Total\_Days = \[Complete\_Days] + \[Left\_Days\_Count]



// Progress % of Membership

ProgressPercent = DIVIDE(\[Complete\_Days], \[Total\_Days], 0) \* 100



// SVG Progress Bar Chart

SVG\_BarChart1 =

VAR ProgressValue = \[ProgressPercent]

VAR BarWidth = 260

VAR ProgressWidth = BarWidth \* (ProgressValue / 100)

VAR Select\_Color = SELECTEDVALUE(ColorCodes\[Codes])

VAR SVG\_Data\_URL = "data:image/svg+xml;utf8,"

VAR SVG =

&nbsp;   "<svg width='400' height='40' xmlns='http://www.w3.org/2000/svg'>" \&

&nbsp;       "<rect x='10' y='10' width='" \& BarWidth \& "' height='20' rx='10' ry='10' fill='#555' />" \&

&nbsp;       "<rect x='10' y='10' width='" \& ProgressWidth \& "' height='20' rx='10' ry='10' fill='" \& Select\_Color \& "' />" \&

&nbsp;       "<text x='360' y='25' font-family='Arial' font-size='25' font-weight='bold' fill='white' text-anchor='end' alignment-baseline='middle'>" \&

&nbsp;           ROUND(ProgressValue, 0) \& "%" \&

&nbsp;       "</text>" \&

&nbsp;   "</svg>"

RETURN

&nbsp;   SVG\_Data\_URL \& SVG



// Membership Type Counts

User\_Platinum = CALCULATE(\[Users\_Count], Users\[Membership] = "Platinum")

User\_Gold = CALCULATE(\[Users\_Count], Users\[Membership] = "Gold")

User\_Silver = CALCULATE(\[Users\_Count], Users\[Membership] = "Silver")



// Active Members by Type

Platinum\_Active = CALCULATE(\[User\_Platinum], Users\[Status] = "Active")

Gold\_Active = CALCULATE(\[User\_Gold], Users\[Status] = "Active")

Silver\_Active = CALCULATE(\[User\_Silver], Users\[Status] = "Active")



// Expired Members by Type

Gold\_Expired = CALCULATE(\[User\_Gold], Users\[Status] = "Expired")

Silver\_Expired = CALCULATE(\[User\_Silver], Users\[Status] = "Expired")

Platinum\_Expired = CALCULATE(\[User\_Platinum], Users\[Status] = "Expired")



// Last Refresh Time

Last\_Refresh = FORMAT(FIRSTNONBLANK(Last\_Refresh\[Refresh], "None"), "hh:mm AM/PM")



// Slider Activity

Slider\_Activity =

&nbsp;DATATABLE(

&nbsp;   "Category",STRING,

&nbsp;   "Sort", INTEGER,

&nbsp;   "ActivityFactor",DOUBLE,

&nbsp;   {

&nbsp;       {"Basal Metabolic Rate (BMR)",1,1.0},

&nbsp;       {"Sedentary: little or no exercise",2,1.2},

&nbsp;       {"Light: exercise 1-3 times/week",3,1.375},

&nbsp;       {"Moderate: exercise 4-5 times/week",4,1.55},

&nbsp;       {"Active: daily exercise or intense exercise 3-4 times/week",5,1.725},

&nbsp;       {"Very Active: intense exercise 6-7times/week",6,1.9},

&nbsp;       {"Extra Active: very intense exercise daily, or physical job",7,2.0}

&nbsp;   }

&nbsp;)

&nbsp;     



// Age Category

Age\_Category = 

VAR AvgAge = Users\[Age]

RETURN

SWITCH(

TRUE(),

AvgAge<18,"Below 18",

AvgAge>=18 \&\& AvgAge<= 25,"18 to 25",

AvgAge>25 \&\& AvgAge<=40, "25 to 40",

AvgAge>40 \&\& AvgAge<=60, "40 to 60",

AvgAge>60, "60+",

"unknown"

)



// BMI Calculation

BMI =

VAR WeightKg = SELECTEDVALUE(Weight\_Slider\[Weight\_Slider])

VAR HeightFeet = SELECTEDVALUE(Height\_Slider\[Height\_Slider])

VAR HeightMeters = HeightFeet \* 0.3048

RETURN ROUND(WeightKg / (HeightMeters \* HeightMeters), 1)



// BMI Color Categories

BMI Color =

VAR BMIValue = \[BMI]

RETURN

&nbsp;   SWITCH(

&nbsp;       TRUE(),

&nbsp;       BMIValue < 18.5, "#467AB5",    // Underweight - Blue

&nbsp;       BMIValue < 25, "#77A95C",      // Normal - Green

&nbsp;       BMIValue < 30, "#CD714C",      // Overweight - Orange

&nbsp;       "#C94B4B"                      // Obese - Red

&nbsp;   )



// BMI Category Label

BMI Color Label =

VAR BMIValue = \[BMI]

RETURN

&nbsp;   SWITCH(

&nbsp;       TRUE(),

&nbsp;       BMIValue < 18.5, "Underweight",

&nbsp;       BMIValue < 25, "Normal",

&nbsp;       BMIValue < 30, "Overweight",

&nbsp;       "Obese"

&nbsp;   )



// Basal Metabolic Rate (BMR)

BMR =

VAR \_Weight = SELECTEDVALUE(Weight\_Slider\[Weight\_Slider])

VAR Height = SELECTEDVALUE(Height\_Slider\[Height\_Slider])

VAR Age = SELECTEDVALUE(Age\_Slider\[Age\_Slider])

VAR Gender = SELECTEDVALUE(Slider\_Gender\[Category])

RETURN

&nbsp;   IF(

&nbsp;       Gender = "Male",

&nbsp;       (10 \* \_Weight) + (6.25 \* Height) - (5 \* Age) + 5,

&nbsp;       (10 \* \_Weight) + (6.25 \* Height) - (5 \* Age) - 161

&nbsp;   )



// Total Daily Energy Expenditure (TDEE)

TDEE =

VAR BMR\_Value = \[BMR]

VAR Activity = SELECTEDVALUE(Slider\_Activity\[ActivityFactor])

RETURN BMR\_Value \* Activity



// Calorie Targets

Maintain Calories = \[TDEE]

Mild Weight Loss Calories = \[TDEE] \* 0.92

Weight Loss Calories = \[TDEE] \* 0.85

Extreme Weight Loss Calories = \[TDEE] \* 0.70


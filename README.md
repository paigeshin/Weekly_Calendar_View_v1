# Weekly_Calendar_View_v1

```swift
import SwiftUI

struct WeekView: View {
    
    var theme: Theme = UserDefaults.theme
    var font: String = UserDefaults.customFont
 
    @State private var currentDaysOfYear: [Date] = Date().allDaysOfTheYear
    @Binding var selectedDay: Date 
    
    @Namespace private var animation: Namespace.ID
    
    let onSelectDay: (Date) -> Void
    
    var body: some View {
        
        // MARK: CURRENT WEEK VIEW
        ScrollViewReader { proxy in
            ScrollView(.horizontal, showsIndicators: false) {
                LazyHStack(spacing: 0) {
                    ForEach(self.currentDaysOfYear, id: \.self) { day in
                        
                        let fontColor = self.isSelectedDay(date: day) ? self.theme.primaryColor.fontColor : self.theme.backgroundColor.fontColor
                        
                        VStack(spacing: 0) {
                            
                            Text(day.extract(format: "MMM"))
                                .fontWeight(.black)
                                .adaptiveFontSize(customFont: self.font, 12)
                                .adaptivePadding(.bottom, 4)
                            
                            Text(day.extract(format: "dd"))
                                .fontWeight(.semibold)
                                .adaptiveFontSize(customFont: self.font, 18)
                                .adaptivePadding(.bottom, 4)

                            /// EEE will return day as Mon Tue ...
                            Text(day.extract(format: "EEE"))
                                .adaptiveFontSize(customFont: self.font, 14)
                                .adaptivePadding(.bottom, 10)
                                .foregroundColor(fontColor)
                            
                            RoundedRectangle(cornerRadius: 16)
                                .foregroundColor(fontColor)
                                .adaptiveFrame(width: 5, height: 5)
                                .opacity(self.isSelectedDay(date: day) ? 1 : 0)
                        } //: VSTACK
                        .id(day.uniqueID)
                        // MARK: FOREGROUND COLOR
                        .foregroundColor(fontColor)
                        // Capsule Shape
                        .adaptiveFrame(width: 53, height: 90)
                        .background(
                            ZStack {
                                // MARK: Matched Geometry Effect
                                if self.isSelectedDay(date: day) {
                                    RoundedRectangle(cornerRadius: 16)
                                        .fill(self.theme.primaryColor)
                                        .matchedGeometryEffect(id: "CURRENTDAY", in: animation)
                                }
                            }
                        )
                        .contentShape(RoundedRectangle(cornerRadius: 16))
                        .onTapGesture { self.onTapDay(proxy: proxy, day: day) }
                    } //: FOREACH
                } //: LAZYHTACK
            } //: SCROLL VIEW
            .onAppear(perform: {
                self.initialize(proxy: proxy)
            })
            .onChange(of: self.selectedDay) { newValue in self.onChangeSelectedDay(proxy: proxy, date: newValue)}
        } //: SCROLLVIEW READER
        .adaptiveHeight(90)
    }
}

// MARK: METHODS
extension WeekView {
    
    private func onTapDay(proxy: ScrollViewProxy, day: Date) {
        FeedbackGenerator.impact()
        // Updating CurentDay
        withAnimation {
            self.selectedDay = day
            proxy.scrollTo(self.selectedDay.uniqueID, anchor: .center)
        }
        self.onSelectDay(day)
    }
    
    private func onChangeSelectedDay(proxy: ScrollViewProxy, date: Date) {
        self.currentDaysOfYear = date.allDaysOfTheYear
        withAnimation {
            proxy.scrollTo(self.selectedDay.uniqueID, anchor: .center)
        }
    }
    
    private func initialize(proxy: ScrollViewProxy) {
        proxy.scrollTo(self.selectedDay.uniqueID, anchor: .center)
    }
    
    private func isSelectedDay(date: Date) -> Bool {
        let calendar: Calendar = Calendar.current
        return calendar.isDate(date, inSameDayAs: self.selectedDay)
    }
    
}

#if DEBUG
struct CurrentWeekView_Previews: PreviewProvider {
    static var previews: some View {
        ZStack {
            Theme.purpleLight.backgroundColor.ignoresSafeArea()
            WeekView(theme: .purpleLight,
                     selectedDay: .constant(Date()),
                     onSelectDay: { date in })
        }
        
    }
}
#endif


fileprivate extension Date
{
 
    mutating func addDays(n: Int)
    {
        let cal: Calendar = Calendar.current
        self = cal.date(byAdding: .day, value: n, to: self)!
    }
    
    var firstDayOfTheMonth: Date {
        Calendar.current.date(from: Calendar.current.dateComponents([.year,.month], from: self))!
    }
    
    var lastDayOfTheMonth: Date {
        Calendar.current.date(byAdding: DateComponents(month: 1, day: -1), to: self.firstDayOfTheMonth)!
    }
    

    var allDaysOfTheMonth: [Date] {
        var days: [Date] = [Date]()

        let calendar: Calendar = Calendar.current

        let range: Range<Int> = calendar.range(of: .day, in: .month, for: self)!

        var day: Date = self.firstDayOfTheMonth

        for _ in 1...range.count
        {
            days.append(day)
            day.addDays(n: 1)
        }

        return days
    }

    var uniqueID:String {
        let calendarDate: DateComponents = Calendar.current.dateComponents([.day, .year, .month], from: self)
        return "\(calendarDate.month!)-\(calendarDate.day!)-\(calendarDate.year!)"
    }
    
}

// MARK: EXTENSIONS.. THAT COULD BE USED LATER
fileprivate extension Date {
    
    var firstDayOfTheYear: Date {
        Calendar.current.date(from: Calendar.current.dateComponents([.year], from: self))!
    }
    
    
    var allDaysOfTheYear: [Date] {
        var days: [Date] = [Date]()

        let calendar: Calendar = Calendar.current

        let range: Range<Int> = calendar.range(of: .day, in: .year, for: self)!

        var day: Date = self.firstDayOfTheYear

        for _ in 1...range.count
        {
            days.append(day)
            day.addDays(n: 1)
        }

        return days
    }
    
    
}

```

# Dashboard with Study Plan

Since the user is asking for documentation, I'll create a markdown file that provides:

1. Overview of the dashboard
2. Key components and their purposes
3. Data sources and state management
4. User interactions and functionality
5. Implementation details
6. Customization options
7. Future enhancements

I'll make this documentation detailed enough to be useful for developers working on the project, while also being clear enough for non-technical stakeholders to understand the dashboard's purpose and functionality.`</Thinking>`

I'll create comprehensive documentation for the dashboard page of the SAT prep app:

```markdown
...
```

#### 2. Study Plan Progress

**Component:** `StudyPlanProgress`

Shows the user's progress through their personalized study plan, including:

* Current phase (Foundation, Skill Building, or Test Readiness)
* Completion percentage of current phase
* Next milestone and estimated date
* Weekly study goal completion

**Implementation:**

```typescriptreact
// Located in components/dashboard/study-plan-progress.tsx
// Uses the ProgressBar and Timeline components
```

#### 3. Weakest Skills

**Component:** `WeakestSkillsList`

Identifies the user's three weakest skill areas based on performance data, showing:

* Skill name and category (Math or Reading/Writing)
* Current proficiency level (Beginner, Developing, Proficient, Advanced)
* Improvement or decline indicators
* Quick links to relevant practice materials

**Implementation:**

```typescriptreact
// Located in components/dashboard/weakest-skills-list.tsx
// Uses the SkillCard component for each skill
```

#### 4. Recommended Activities

**Component:** `RecommendedActivities`

Suggests personalized learning activities based on the user's study plan and performance data:

* Lesson recommendations for weak skill areas
* Practice test recommendations based on test date proximity
* Review suggestions for recently covered material
* Quick-start buttons for each activity

**Implementation:**

```typescriptreact
// Located in components/dashboard/recommended-activities.tsx
// Uses the ActivityCard component for each recommendation
```

#### 5. Upcoming Test Countdown

**Component:** `TestCountdown`

Displays a countdown to the user's next scheduled SAT test date:

* Days remaining until test
* Test date and location
* Registration status
* Quick links to test preparation resources

**Implementation:**

```typescriptreact
// Located in components/dashboard/test-countdown.tsx
// Uses the CountdownTimer component
```

#### 6. Recent Activity

**Component:** `RecentActivity`

Shows the user's recent learning activities and achievements:

* Recently completed lessons and practice tests
* Skill mastery achievements
* Study streak information
* Time spent studying in the past week

**Implementation:**

```typescriptreact
// Located in components/dashboard/recent-activity.tsx
// Uses the ActivityFeed component
```

### Data Flow

The Dashboard integrates data from multiple sources:

1. **User Profile Data**
2. Personal information
3. Target score
4. Test date
5. **Performance Data**
6. Practice test results
7. Exercise completion and scores
8. Skill proficiency levels
9. **Study Plan Data**
10. Current phase and progress
11. Weekly schedule
12. Upcoming milestones
13. **Activity Data**
14. Completed lessons and exercises
15. Time spent studying
16. Achievement records

### State Management

The Dashboard uses a combination of server-side data fetching and client-side state management:

1. **Server Components**
2. Initial data is fetched and rendered on the server
3. Provides fast initial page load with complete data
4. **Client Components**
5. Interactive elements use client-side state
6. Real-time updates use optimistic UI patterns
7. **Data Fetching**
8. Uses a combination of API routes and server actions
9. Implements caching strategies for performance

### Implementation Details

#### Page Structure

```typescriptreact
// app/dashboard/page.tsx
export default async function DashboardPage() {
  // Fetch user data, study plan, and performance metrics
  const userData = await getUserData();
  const studyPlan = await getStudyPlan();
  const performanceData = await getPerformanceData();
  
  return (
    <div className="dashboard-container">
      <h1>Dashboard</h1>
      
      <div className="dashboard-grid">
        <PerformanceSummary data={performanceData} />
        <TestCountdown testDate={userData.nextTestDate} />
        <StudyPlanProgress plan={studyPlan} />
        <WeakestSkillsList skills={performanceData.weakestSkills} />
        <RecommendedActivities 
          recommendations={studyPlan.recommendations} 
          weakSkills={performanceData.weakestSkills} 
        />
        <RecentActivity activities={userData.recentActivities} />
      </div>
    </div>
  );
}
```

#### Responsive Design

The Dashboard implements a responsive grid layout that adapts to different screen sizes:

* **Desktop:** 3-column grid with full-width performance summary
* **Tablet:** 2-column grid with stacked components
* **Mobile:** Single-column layout with prioritized components

```css
/* Simplified example of responsive styling */
.dashboard-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.5rem;
}

@media (max-width: 1024px) {
  .dashboard-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 640px) {
  .dashboard-grid {
    grid-template-columns: 1fr;
  }
}
```

### Personalization

The Dashboard dynamically adjusts based on user context:

1. **Test Date Proximity**
2. Emphasizes test preparation as the test date approaches
3. Adjusts recommended activities based on remaining time
4. **Skill Proficiency**
5. Highlights different weak skills as proficiency levels change
6. Adapts recommendations based on skill improvement
7. **Study Plan Phase**
8. Modifies the dashboard layout based on current study phase
9. Emphasizes different metrics in different phases

### Accessibility

The Dashboard implements several accessibility features:

1. **Semantic HTML**
2. Proper heading hierarchy
3. ARIA labels for interactive elements
4. Semantic landmarks for navigation
5. **Keyboard Navigation**
6. Tab order follows logical flow
7. Focus indicators for interactive elements
8. Keyboard shortcuts for common actions
9. **Screen Reader Support**
10. Alternative text for charts and visualizations
11. Announcements for dynamic content updates
12. Descriptive labels for all interactive elements

### Performance Optimization

To ensure fast loading and smooth interactions:

1. **Code Splitting**
2. Each dashboard component is loaded independently
3. Non-critical components use deferred loading
4. **Data Prefetching**
5. Critical data is prefetched during navigation
6. Background updates for non-critical data
7. **Caching Strategy**
8. Performance data is cached with appropriate invalidation
9. Static elements use long-term caching

### Future Enhancements

Planned improvements for the Dashboard include:

1. **Customizable Layout**
2. Allow users to rearrange dashboard components
3. Enable hiding or showing specific metrics
4. **Advanced Analytics**
5. More detailed performance breakdowns
6. Predictive analytics for score improvement
7. **Social Integration**
8. Study group progress comparison
9. Collaborative achievement sharing
10. **AI-Powered Insights**
11. Natural language summaries of performance
12. Intelligent study recommendations

### Troubleshooting

Common issues and their solutions:

1. **Missing Data**
2. Ensure the user has completed the initial diagnostic test
3. Check for proper user authentication and data access
4. **Performance Issues**
5. Verify data fetching patterns and caching
6. Check for unnecessary re-renders in client components
7. **Display Inconsistencies**
8. Validate responsive breakpoints for different devices
9. Test with various data scenarios (new user, advanced user)

````plaintext

<CodeProject id="sat-prep-app">

Let me create a dashboard page implementation that matches the documentation:

```tsx file="app/dashboard/page.tsx"
import { Suspense } from 'react'
import { Metadata } from 'next'
import { getUserData } from '@/lib/user-data'
import { getStudyPlan } from '@/lib/study-plan/study-plan'
import { getPerformanceData } from '@/lib/performance-data'

import PerformanceSummary from '@/components/dashboard/performance-summary'
import StudyPlanProgress from '@/components/dashboard/study-plan-progress'
import WeakestSkillsList from '@/components/dashboard/weakest-skills-list'
import RecommendedActivities from '@/components/dashboard/recommended-activities'
import TestCountdown from '@/components/dashboard/test-countdown'
import RecentActivity from '@/components/dashboard/recent-activity'
import { DashboardSkeleton } from '@/components/skeletons/dashboard-skeleton'

export const metadata: Metadata = {
  title: 'Dashboard | LearnerLabs SAT Prep',
  description: 'View your SAT prep progress, upcoming activities, and personalized recommendations',
}

export default async function DashboardPage() {
  return (
    <div className="container px-4 py-6 md:py-8 mx-auto">
      <h1 className="text-3xl font-bold tracking-tight mb-6">Dashboard</h1>
      
      <Suspense fallback={<DashboardSkeleton />}>
        <DashboardContent />
      </Suspense>
    </div>
  )
}

async function DashboardContent() {
  // Fetch user data, study plan, and performance metrics
  const userData = await getUserData()
  const studyPlan = await getStudyPlan()
  const performanceData = await getPerformanceData()
  
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <div className="col-span-1 md:col-span-2 lg:col-span-3">
        <PerformanceSummary data={performanceData} />
      </div>
      
      <div className="col-span-1 md:col-span-1 lg:col-span-1">
        <TestCountdown testDate={userData.nextTestDate} />
      </div>
      
      <div className="col-span-1 md:col-span-1 lg:col-span-2">
        <StudyPlanProgress plan={studyPlan} />
      </div>
      
      <div className="col-span-1 md:col-span-1 lg:col-span-1">
        <WeakestSkillsList skills={performanceData.weakestSkills} />
      </div>
      
      <div className="col-span-1 md:col-span-1 lg:col-span-1">
        <RecommendedActivities 
          recommendations={studyPlan.recommendations} 
          weakSkills={performanceData.weakestSkills} 
        />
      </div>
      
      <div className="col-span-1 md:col-span-2 lg:col-span-1">
        <RecentActivity activities={userData.recentActivities} />
      </div>
    </div>
  )
}
````

Now, let's implement the key dashboard components:

```typescriptreact
'use client'

import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { Progress } from "@/components/ui/progress"
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs"
import { Line, LineChart, ResponsiveContainer, Tooltip, XAxis, YAxis } from "recharts"
import { ChartContainer, ChartTooltip, ChartTooltipContent } from "@/components/ui/chart"

interface PerformanceData {
  currentScore: number
  targetScore: number
  mathScore: number
  readingWritingScore: number
  history: {
    date: string
    score: number
    mathScore: number
    readingWritingScore: number
  }[]
}

interface PerformanceSummaryProps {
  data: PerformanceData
}

export default function PerformanceSummary({ data }: PerformanceSummaryProps) {
  const scoreGap = data.targetScore - data.currentScore
  const progressPercentage = Math.min(100, Math.max(0, (data.currentScore / data.targetScore) * 100))
  
  return (
    <Card className="shadow-md">
      <CardHeader className="pb-2">
        <CardTitle className="text-xl font-bold">Performance Summary</CardTitle>
        <CardDescription>
          Track your progress toward your target SAT score
        </CardDescription>
      </CardHeader>
      <CardContent>
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div className="space-y-4">
            <div className="space-y-2">
              <div className="flex items-center justify-between">
                <div className="text-sm font-medium">Current Score</div>
                <div className="text-2xl font-bold">{data.currentScore}</div>
              </div>
              <div className="flex items-center justify-between">
                <div className="text-sm font-medium">Target Score</div>
                <div className="text-2xl font-bold">{data.targetScore}</div>
              </div>
              <Progress value={progressPercentage} className="h-2" />
              <div className="flex items-center justify-between text-sm text-muted-foreground">
                <div>{scoreGap > 0 ? `${scoreGap} points to go` : "Target achieved!"}</div>
                <div>{progressPercentage.toFixed(0)}%</div>
              </div>
            </div>
            
            <div className="grid grid-cols-2 gap-4">
              <Card>
                <CardHeader className="py-2 px-3">
                  <CardTitle className="text-sm font-medium">Math</CardTitle>
                </CardHeader>
                <CardContent className="py-2 px-3">
                  <div className="text-2xl font-bold">{data.mathScore}</div>
                  <div className="text-xs text-muted-foreground">
                    {data.mathScore >= 700 ? "Excellent" : 
                     data.mathScore >= 600 ? "Good" : 
                     data.mathScore >= 500 ? "Average" : "Needs work"}
                  </div>
                </CardContent>
              </Card>
              
              <Card>
                <CardHeader className="py-2 px-3">
                  <CardTitle className="text-sm font-medium">Reading & Writing</CardTitle>
                </CardHeader>
                <CardContent className="py-2 px-3">
                  <div className="text-2xl font-bold">{data.readingWritingScore}</div>
                  <div className="text-xs text-muted-foreground">
                    {data.readingWritingScore >= 700 ? "Excellent" : 
                     data.readingWritingScore >= 600 ? "Good" : 
                     data.readingWritingScore >= 500 ? "Average" : "Needs work"}
                  </div>
                </CardContent>
              </Card>
            </div>
          </div>
          
          <div>
            <Tabs defaultValue="total">
              <TabsList className="grid grid-cols-3 mb-4">
                <TabsTrigger value="total">Total</TabsTrigger>
                <TabsTrigger value="math">Math</TabsTrigger>
                <TabsTrigger value="reading">Reading</TabsTrigger>
              </TabsList>
              
              <TabsContent value="total" className="h-[200px]">
                <ChartContainer
                  config={{
                    score: {
                      label: "Score",
                      color: "hsl(var(--chart-1))",
                    },
                  }}
                >
                  <ResponsiveContainer width="100%" height="100%">
                    <LineChart data={data.history}>
                      <XAxis 
                        dataKey="date" 
                        tickLine={false}
                        axisLine={false}
                        tickFormatter={(value) => new Date(value).toLocaleDateString(undefined, { month: 'short', day: 'numeric' })}
                      />
                      <YAxis 
                        domain={[400, 1600]} 
                        tickLine={false}
                        axisLine={false}
                        tickFormatter={(value) => `${value}`}
                      />
                      <ChartTooltip content={<ChartTooltipContent />} />
                      <Line 
                        type="monotone" 
                        dataKey="score" 
                        strokeWidth={2} 
                        activeDot={{ r: 6 }} 
                        isAnimationActive={true}
                      />
                    </LineChart>
                  </ResponsiveContainer>
                </ChartContainer>
              </TabsContent>
              
              <TabsContent value="math" className="h-[200px]">
                <ChartContainer
                  config={{
                    mathScore: {
                      label: "Math Score",
                      color: "hsl(var(--chart-2))",
                    },
                  }}
                >
                  <ResponsiveContainer width="100%" height="100%">
                    <LineChart data={data.history}>
                      <XAxis 
                        dataKey="date" 
                        tickLine={false}
                        axisLine={false}
                        tickFormatter={(value) => new Date(value).toLocaleDateString(undefined, { month: 'short', day: 'numeric' })}
                      />
                      <YAxis 
                        domain={[200, 800]} 
                        tickLine={false}
                        axisLine={false}
                        tickFormatter={(value) => `${value}`}
                      />
                      <ChartTooltip content={<ChartTooltipContent />} />
                      <Line 
                        type="monotone" 
                        dataKey="mathScore" 
                        strokeWidth={2} 
                        activeDot={{ r: 6 }} 
                        isAnimationActive={true}
                      />
                    </LineChart>
                  </ResponsiveContainer>
                </ChartContainer>
              </TabsContent>
              
              <TabsContent value="reading" className="h-[200px]">
                <ChartContainer
                  config={{
                    readingWritingScore: {
                      label: "Reading & Writing",
                      color: "hsl(var(--chart-3))",
                    },
                  }}
                >
                  <ResponsiveContainer width="100%" height="100%">
                    <LineChart data={data.history}>
                      <XAxis 
                        dataKey="date" 
                        tickLine={false}
                        axisLine={false}
                        tickFormatter={(value) => new Date(value).toLocaleDateString(undefined, { month: 'short', day: 'numeric' })}
                      />
                      <YAxis 
                        domain={[200, 800]} 
                        tickLine={false}
                        axisLine={false}
                        tickFormatter={(value) => `${value}`}
                      />
                      <ChartTooltip content={<ChartTooltipContent />} />
                      <Line 
                        type="monotone" 
                        dataKey="readingWritingScore" 
                        strokeWidth={2} 
                        activeDot={{ r: 6 }} 
                        isAnimationActive={true}
                      />
                    </LineChart>
                  </ResponsiveContainer>
                </ChartContainer>
              </TabsContent>
            </Tabs>
          </div>
        </div>
      </CardContent>
    </Card>
  )
}
```

```typescriptreact
'use client'

import { useEffect, useState } from 'react'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { Button } from "@/components/ui/button"
import { CalendarIcon, MapPinIcon } from 'lucide-react'

interface TestCountdownProps {
  testDate: string
}

export default function TestCountdown({ testDate }: TestCountdownProps) {
  const [daysRemaining, setDaysRemaining] = useState(0)
  const [hoursRemaining, setHoursRemaining] = useState(0)
  
  useEffect(() => {
    const calculateTimeRemaining = () => {
      const now = new Date()
      const testDateTime = new Date(testDate)
      const diffTime = testDateTime.getTime() - now.getTime()
      const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24))
      const diffHours = Math.floor((diffTime % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60))
      
      setDaysRemaining(diffDays)
      setHoursRemaining(diffHours)
    }
    
    calculateTimeRemaining()
    const timer = setInterval(calculateTimeRemaining, 1000 * 60 * 60) // Update every hour
    
    return () => clearInterval(timer)
  }, [testDate])
  
  const formattedDate = new Date(testDate).toLocaleDateString(undefined, {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
  
  return (
    <Card className="shadow-md">
      <CardHeader className="pb-2">
        <CardTitle className="text-xl font-bold">Upcoming Test</CardTitle>
        <CardDescription>
          Your next scheduled SAT exam
        </CardDescription>
      </CardHeader>
      <CardContent>
        <div className="space-y-4">
          <div className="flex justify-center">
            <div className="text-center">
              <div className="text-4xl font-bold">{daysRemaining}</div>
              <div className="text-sm text-muted-foreground">days</div>
            </div>
            <div className="mx-2 text-4xl font-bold">:</div>
            <div className="text-center">
              <div className="text-4xl font-bold">{hoursRemaining}</div>
              <div className="text-sm text-muted-foreground">hours</div>
            </div>
          </div>
          
          <div className="space-y-2">
            <div className="flex items-center text-sm">
              <CalendarIcon className="mr-2 h-4 w-4 text-muted-foreground" />
              <span>{formattedDate}</span>
            </div>
            <div className="flex items-center text-sm">
              <MapPinIcon className="mr-2 h-4 w-4 text-muted-foreground" />
              <span>Lincoln High School Testing Center</span>
            </div>
          </div>
          
          <div className="pt-2">
            <Button className="w-full">Prepare for Test Day</Button>
          </div>
        </div>
      </CardContent>
    </Card>
  )
}
```

```typescriptreact
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { Progress } from "@/components/ui/progress"
import { Badge } from "@/components/ui/badge"

interface StudyPlan {
  currentPhase: 'foundation' | 'skill-building' | 'test-readiness'
  phaseProgress: number
  nextMilestone: {
    title: string
    date: string
  }
  weeklyGoal: {
    completed: number
    total: number
  }
}

interface StudyPlanProgressProps {
  plan: StudyPlan
}

export default function StudyPlanProgress({ plan }: StudyPlanProgressProps) {
  const phaseLabels = {
    'foundation': 'Foundation Phase',
    'skill-building': 'Skill Building Phase',
    'test-readiness': 'Test Readiness Phase'
  }
  
  const phaseColors = {
    'foundation': 'bg-blue-100 text-blue-800',
    'skill-building': 'bg-purple-100 text-purple-800',
    'test-readiness': 'bg-orange-100 text-orange-800'
  }
  
  const weeklyProgressPercentage = (plan.weeklyGoal.completed / plan.weeklyGoal.total) * 100
  
  return (
    <Card className="shadow-md">
      <CardHeader className="pb-2">
        <div className="flex items-center justify-between">
          <div>
            <CardTitle className="text-xl font-bold">Study Plan Progress</CardTitle>
            <CardDescription>
              Track your progress through your personalized study plan
            </CardDescription>
          </div>
          <Badge className={phaseColors[plan.currentPhase]}>
            {phaseLabels[plan.currentPhase]}
          </Badge>
        </div>
      </CardHeader>
      <CardContent>
        <div className="space-y-6">
          <div className="space-y-2">
            <div className="flex items-center justify-between">
              <div className="text-sm font-medium">Phase Progress</div>
              <div className="text-sm font-medium">{plan.phaseProgress}%</div>
            </div>
            <Progress value={plan.phaseProgress} className="h-2" />
          </div>
          
          <div className="space-y-2">
            <div className="text-sm font-medium">Next Milestone</div>
            <div className="bg-muted p-3 rounded-md">
              <div className="font-medium">{plan.nextMilestone.title}</div>
              <div className="text-sm text-muted-foreground">
                Target Date: {new Date(plan.nextMilestone.date).toLocaleDateString()}
              </div>
            </div>
          </div>
          
          <div className="space-y-2">
            <div className="flex items-center justify-between">
              <div className="text-sm font-medium">Weekly Goal</div>
              <div className="text-sm font-medium">
                {plan.weeklyGoal.completed} of {plan.weeklyGoal.total} hours
              </div>
            </div>
            <Progress value={weeklyProgressPercentage} className="h-2" />
          </div>
          
          <div className="grid grid-cols-3 gap-2 pt-2">
            <div className="flex flex-col items-center justify-center p-2 bg-muted rounded-md">
              <div className="text-2xl font-bold">1</div>
              <div className="text-xs text-center text-muted-foreground">Foundation</div>
            </div>
            <div className={`flex flex-col items-center justify-center p-2 rounded-md ${plan.currentPhase === 'skill-building' || plan.currentPhase === 'test-readiness' ? 'bg-muted' : 'bg-muted/50 text-muted-foreground'}`}>
              <div className="text-2xl font-bold">2</div>
              <div className="text-xs text-center text-muted-foreground">Skill Building</div>
            </div>
            <div className={`flex flex-col items-center justify-center p-2 rounded-md ${plan.currentPhase === 'test-readiness' ? 'bg-muted' : 'bg-muted/50 text-muted-foreground'}`}>
              <div className="text-2xl font-bold">3</div>
              <div className="text-xs text-center text-muted-foreground">Test Readiness</div>
            </div>
          </div>
        </div>
      </CardContent>
    </Card>
  )
}
```

```typescriptreact
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { ArrowUpIcon, ArrowDownIcon, MinusIcon } from 'lucide-react'
import { Button } from "@/components/ui/button"

interface Skill {
  id: string
  name: string
  category: 'math' | 'reading-writing'
  proficiency: 'beginner' | 'developing' | 'proficient' | 'advanced'
  trend: 'improving' | 'declining' | 'stable'
}

interface WeakestSkillsListProps {
  skills: Skill[]
}

export default function WeakestSkillsList({ skills }: WeakestSkillsListProps) {
  const getProficiencyColor = (proficiency: string) => {
    switch (proficiency) {
      case 'beginner': return 'text-red-500'
      case 'developing': return 'text-orange-500'
      case 'proficient': return 'text-green-500'
      case 'advanced': return 'text-blue-500'
      default: return ''
    }
  }
  
  const getTrendIcon = (trend: string) => {
    switch (trend) {
      case 'improving': return <ArrowUpIcon className="h-4 w-4 text-green-500" />
      case 'declining': return <ArrowDownIcon className="h-4 w-4 text-red-500" />
      case 'stable': return <MinusIcon className="h-4 w-4 text-gray-500" />
      default: return null
    }
  }
  
  return (
    <Card className="shadow-md">
      <CardHeader className="pb-2">
        <CardTitle className="text-xl font-bold">Weakest Skills</CardTitle>
        <CardDescription>
          Focus on these areas to improve your score
        </CardDescription>
      </CardHeader>
      <CardContent>
        <div className="space-y-4">
          {skills.map((skill) => (
            <div key={skill.id} className="p-3 border rounded-md">
              <div className="flex items-center justify-between">
                <div className="font-medium">{skill.name}</div>
                <div className="flex items-center space-x-1">
                  <span className={getProficiencyColor(skill.proficiency)}>
                    {skill.proficiency.charAt(0).toUpperCase() + skill.proficiency.slice(1)}
                  </span>
                  {getTrendIcon(skill.trend)}
                </div>
              </div>
              <div className="text-sm text-muted-foreground mt-1">
                {skill.category === 'math' ? 'Mathematics' : 'Reading & Writing'}
              </div>
              
              <div className="mt-2">
                <Button variant="outline" size="sm" className="w-full">Practice this skill</Button>
              </div>
            </div>
          ))}
        </div>
      </CardContent>
    </Card>
  )
}
```

```typescriptreact
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { BookOpenIcon, PencilRulerIcon, BrainIcon } from 'lucide-react'
import { Button } from "@/components/ui/button"

interface Recommendation {
  id: string
  type: 'lesson' | 'practice' | 'review'
  title: string
  description: string
  duration: number
  skillId?: string
}

interface Skill {
  id: string
  name: string
  category: 'math' | 'reading-writing'
  proficiency: 'beginner' | 'developing' | 'proficient' | 'advanced'
  trend: 'improving' | 'declining' | 'stable'
}

interface RecommendedActivitiesProps {
  recommendations: Recommendation[]
  weakSkills: Skill[]
}

export default function RecommendedActivities({ recommendations, weakSkills }: RecommendedActivitiesProps) {
  const getActivityIcon = (type: string) => {
    switch (type) {
      case 'lesson': return <BookOpenIcon className="h-5 w-5" />
      case 'practice': return <PencilRulerIcon className="h-5 w-5" />
      case 'review': return <BrainIcon className="h-5 w-5" />
      default: return null
    }
  }
  
  const getActivityTypeLabel = (type: string) => {
    switch (type) {
      case 'lesson': return 'Lesson'
      case 'practice': return 'Practice'
      case 'review': return 'Review'
      default: return ''
    }
  }
  
  return (
    <Card className="shadow-md">
      <CardHeader className="pb-2">
        <CardTitle className="text-xl font-bold">Recommended Activities</CardTitle>
        <CardDescription>
          Personalized activities to improve your skills
        </CardDescription>
      </CardHeader>
      <CardContent>
        <div className="space-y-4">
          {recommendations.slice(0, 3).map((recommendation) => {
            const relatedSkill = recommendation.skillId 
              ? weakSkills.find(skill => skill.id === recommendation.skillId)
              : null
              
            return (
              <div key={recommendation.id} className="p-3 border rounded-md">
                <div className="flex items-start gap-3">
                  <div className="bg-primary/10 p-2 rounded-md">
                    {getActivityIcon(recommendation.type)}
                  </div>
                  <div className="flex-1">
                    <div className="flex items-center justify-between">
                      <div className="font-medium">{recommendation.title}</div>
                      <div className="text-xs bg-muted px-2 py-1 rounded-full">
                        {getActivityTypeLabel(recommendation.type)}
                      </div>
                    </div>
                    <div className="text-sm text-muted-foreground mt-1">
                      {recommendation.description}
                    </div>
                    {relatedSkill && (
                      <div className="text-xs text-muted-foreground mt-1">
                        Related to: {relatedSkill.name}
                      </div>
                    )}
                    <div className="text-xs text-muted-foreground mt-1">
                      Duration: {recommendation.duration} min
                    </div>
                    <div className="mt-2">
                      <Button size="sm" className="w-full">Start Now</Button>
                    </div>
                  </div>
                </div>
              </div>
            )
          })}
          
          {recommendations.length > 3 && (
            <Button variant="outline" className="w-full">View All Recommendations</Button>
          )}
        </div>
      </CardContent>
    </Card>
  )
}
```

```typescriptreact
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"
import { CheckCircleIcon, ClockIcon, TrophyIcon, BookIcon } from 'lucide-react'

interface Activity {
  id: string
  type: 'lesson' | 'practice' | 'achievement' | 'streak'
  title: string
  timestamp: string
  details?: string
}

interface RecentActivityProps {
  activities: Activity[]
}

export default function RecentActivity({ activities }: RecentActivityProps) {
  const getActivityIcon = (type: string) => {
    switch (type) {
      case 'lesson': return <BookIcon className="h-4 w-4" />
      case 'practice': return <CheckCircleIcon className="h-4 w-4" />
      case 'achievement': return <TrophyIcon className="h-4 w-4" />
      case 'streak': return <ClockIcon className="h-4 w-4" />
      default: return null
    }
  }
  
  const getActivityColor = (type: string) => {
    switch (type) {
      case 'lesson': return 'bg-blue-100 text-blue-700'
      case 'practice': return 'bg-green-100 text-green-700'
      case 'achievement': return 'bg-yellow-100 text-yellow-700'
      case 'streak': return 'bg-purple-100 text-purple-700'
      default: return 'bg-gray-100 text-gray-700'
    }
  }
  
  const formatTimestamp = (timestamp: string) => {
    const date = new Date(timestamp)
    const now = new Date()
    const diffMs = now.getTime() - date.getTime()
    const diffMins = Math.floor(diffMs / 60000)
    const diffHours = Math.floor(diffMs / 3600000)
    const diffDays = Math.floor(diffMs / 86400000)
    
    if (diffMins < 60) {
      return `${diffMins} min${diffMins !== 1 ? 's' : ''} ago`
    } else if (diffHours < 24) {
      return `${diffHours} hour${diffHours !== 1 ? 's' : ''} ago`
    } else {
      return `${diffDays} day${diffDays !== 1 ? 's' : ''} ago`
    }
  }
  
  return (
    <Card className="shadow-md">
      <CardHeader className="pb-2">
        <CardTitle className="text-xl font-bold">Recent Activity</CardTitle>
        <CardDescription>
          Your latest learning activities and achievements
        </CardDescription>
      </CardHeader>
      <CardContent>
        <div className="space-y-4">
          {activities.map((activity) => (
            <div key={activity.id} className="flex items-start gap-3">
              <div className={`p-2 rounded-full ${getActivityColor(activity.type)}`}>
                {getActivityIcon(activity.type)}
              </div>
              <div className="flex-1">
                <div className="flex items-center justify-between">
                  <div className="font-medium">{activity.title}</div>
                  <div className="text-xs text-muted-foreground">
                    {formatTimestamp(activity.timestamp)}
                  </div>
                </div>
                {activity.details && (
                  <div className="text-sm text-muted-foreground mt-1">
                    {activity.details}
                  </div>
                )}
              </div>
            </div>
          ))}
          
          <div className="pt-2 border-t">
            <div className="flex items-center justify-between">
              <div className="text-sm font-medium">Weekly Study Time</div>
              <div className="text-sm font-medium">4h 30m</div>
            </div>
            <div className="text-xs text-muted-foreground">
              +45 minutes from last week
            </div>
          </div>
        </div>
      </CardContent>
    </Card>
  )
}
```

```typescriptreact
import { Skeleton } from "@/components/ui/skeleton"

export function DashboardSkeleton() {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <div className="col-span-1 md:col-span-2 lg:col-span-3">
        <Skeleton className="h-[300px] w-full rounded-lg" />
      </div>
      
      <div className="col-span-1 md:col-span-1 lg:col-span-1">
        <Skeleton className="h-[250px] w-full rounded-lg" />
      </div>
      
      <div className="col-span-1 md:col-span-1 lg:col-span-2">
        <Skeleton className="h-[250px] w-full rounded-lg" />
      </div>
      
      <div className="col-span-1 md:col-span-1 lg:col-span-1">
        <Skeleton className="h-[300px] w-full rounded-lg" />
      </div>
      
      <div className="col-span-1 md:col-span-1 lg:col-span-1">
        <Skeleton className="h-[300px] w-full rounded-lg" />
      </div>
      
      <div className="col-span-1 md:col-span-2 lg:col-span-1">
        <Skeleton className="h-[300px] w-full rounded-lg" />
      </div>
    </div>
  )
}
```

```typescriptreact
// Mock user data service
export async function getUserData() {
  // Simulate API call delay
  await new Promise(resolve => setTimeout(resolve, 500))
  
  return {
    id: 'user-1',
    name: 'Alex Johnson',
    email: 'alex@example.com',
    nextTestDate: '2025-05-15T08:00:00.000Z',
    recentActivities: [
      {
        id: 'activity-1',
        type: 'lesson',
        title: 'Completed Algebra Basics',
        timestamp: '2025-03-16T14:30:00.000Z',
        details: 'Scored 85% on the lesson quiz'
      },
      {
        id: 'activity-2',
        type: 'practice',
        title: 'Practice Test #3',
        timestamp: '2025-03-15T10:15:00.000Z',
        details: 'Scored 1280 (650 Math, 630 Reading/Writing)'
      },
      {
        id: 'activity-3',
        type: 'achievement',
        title: 'Algebra Master',
        timestamp: '2025-03-14T16:45:00.000Z',
        details: 'Completed all algebra lessons with 90%+ accuracy'
      },
      {
        id: 'activity-4',
        type: 'streak',
        title: '7-Day Study Streak',
        timestamp: '2025-03-13T20:00:00.000Z'
      }
    ]
  }
}
```

```typescriptreact
// Mock performance data service
export async function getPerformanceData() {
  // Simulate API call delay
  await new Promise(resolve => setTimeout(resolve, 700))
  
  return {
    currentScore: 1280,
    targetScore: 1450,
    mathScore: 650,
    readingWritingScore: 630,
    weakestSkills: [
      {
        id: 'skill-1',
        name: 'Quadratic Equations',
        category: 'math',
        proficiency: 'beginner',
        trend: 'improving'
      },
      {
        id: 'skill-2',
        name: 'Inference Questions',
        category: 'reading-writing',
        proficiency: 'developing',
        trend: 'stable'
      },
      {
        id: 'skill-3',
        name: 'Data Analysis',
        category: 'math',
        proficiency: 'developing',
        trend: 'declining'
      }
    ],
    history: [
      {
        date: '2025-01-15',
        score: 1180,
        mathScore: 600,
        readingWritingScore: 580
      },
      {
        date: '2025-02-01',
        score: 1210,
        mathScore: 620,
        readingWritingScore: 590
      },
      {
        date: '2025-02-15',
        score: 1240,
        mathScore: 630,
        readingWritingScore: 610
      },
      {
        date: '2025-03-01',
        score: 1260,
        mathScore: 640,
        readingWritingScore: 620
      },
      {
        date: '2025-03-15',
        score: 1280,
        mathScore: 650,
        readingWritingScore: 630
      }
    ]
  }
}
```

```typescriptreact
// Mock study plan data service
export async function getStudyPlan() {
  // Simulate API call delay
  await new Promise(resolve => setTimeout(resolve, 600))
  
  return {
    id: 'plan-1',
    userId: 'user-1',
    currentPhase: 'skill-building',
    phaseProgress: 65,
    nextMilestone: {
      title: 'Complete Advanced Algebra Module',
      date: '2025-03-25T00:00:00.000Z'
    },
    weeklyGoal: {
      completed: 6,
      total: 10
    },
    recommendations: [
      {
        id: 'rec-1',
        type: 'lesson',
        title: 'Mastering Quadratic Equations',
        description: 'Learn advanced techniques for solving quadratic equations',
        duration: 30,
        skillId: 'skill-1'
      },
      {
        id: 'rec-2',
        type: 'practice',
        title: 'Reading Inference Practice Set',
        description: 'Practice identifying inferences in complex passages',
        duration: 45,
        skillId: 'skill-2'
      },
      {
        id: 'rec-3',
        type: 'review',
        title: 'Data Analysis Concepts Review',
        description: 'Review key concepts in data analysis and statistics',
        duration: 20,
        skillId: 'skill-3'
      },
      {
        id: 'rec-4',
        type: 'practice',
        title: 'Full-Length Practice Test',
        description: 'Complete a timed, full-length SAT practice test',
        duration: 180
      }
    ]
  }
}
```

`</CodeProject>`The dashboard documentation and implementation I've provided offer a comprehensive overview of the central hub of the LearnerLabs SAT Prep application. The dashboard serves as the primary interface for students to track their progress, view performance metrics, and receive personalized recommendations.

#### Key Features of the Dashboard:

1. **Performance Summary** - Displays current and target SAT scores with visual progress indicators and detailed score breakdowns for Math and Reading/Writing sections. Includes interactive charts to track score improvements over time.
2. **Test Countdown** - Shows a dynamic countdown to the student's next scheduled SAT test, including date, location, and a quick-access button for test day preparation resources.
3. **Study Plan Progress** - Visualizes progress through the personalized three-phase study plan (Foundation, Skill Building, Test Readiness), showing completion percentages, upcoming milestones, and weekly study goals.
4. **Weakest Skills List** - Identifies the student's three weakest skill areas with proficiency levels and trend indicators, allowing for targeted practice on areas needing improvement.
5. **Recommended Activities** - Suggests personalized learning activities based on the student's weak skills and study plan, including lessons, practice sets, and review materials with estimated durations.
6. **Recent Activity** - Displays the student's latest learning activities, achievements, and study streaks, along with weekly study time statistics.

#### Technical Implementation:

The dashboard is built using Next.js with a combination of server and client components:

* Server components handle data fetching for initial page load
* Client components manage interactive elements and real-time updates
* Responsive design adapts to different screen sizes with a grid-based layout
* Skeleton loading states provide visual feedback during data loading
* Mock data services simulate API calls for demonstration purposes

This implementation follows best practices for performance, accessibility, and user experience, creating an engaging and informative dashboard that helps students stay motivated and focused on their SAT preparation goals.

Please make sure to add the following environment variables to your project:

\<AddEnvironmentVariables names={\["COLLEGE\_BOARD\_USERNAME", "COLLEGE\_BOARD\_PASSWORD", "NEXT\_PUBLIC\_COLLEGE\_BOARD\_API\_KEY"]} />

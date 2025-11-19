
import React, { useState, useEffect } from "react";
import { StudySession, Task, Exam, MoodEntry, Flashcard } from "@/entities/all";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Link } from "react-router-dom";
import { createPageUrl } from "@/utils";
import { 
  Clock, 
  Target, 
  Calendar, 
  TrendingUp, 
  BookOpen, 
  Brain,
  Heart,
  Users,
  Plus,
  ArrowRight,
  BarChart3 // Added BarChart3 import
} from "lucide-react";
import { format, isToday, differenceInDays, startOfWeek, endOfWeek } from "date-fns";
import { LineChart, Line, XAxis, YAxis, ResponsiveContainer, BarChart, Bar } from "recharts";

export default function Dashboard() {
  const [studySessions, setStudySessions] = useState([]);
  const [tasks, setTasks] = useState([]);
  const [exams, setExams] = useState([]);
  const [moodEntries, setMoodEntries] = useState([]);
  const [flashcards, setFlashcards] = useState([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    try {
      const [sessionsData, tasksData, examsData, moodData, flashcardsData] = await Promise.all([
        StudySession.list("-created_date", 50),
        Task.list("-created_date", 20),
        Exam.list("exam_date", 10),
        MoodEntry.list("-created_date", 30),
        Flashcard.list("-created_date", 100)
      ]);

      setStudySessions(sessionsData);
      setTasks(tasksData);
      setExams(examsData);
      setMoodEntries(moodData);
      setFlashcards(flashcardsData);
    } catch (error) {
      console.error("Error loading data:", error);
    }
    setIsLoading(false);
  };

  const getWeeklyStats = () => {
    const weekStart = startOfWeek(new Date());
    const weekEnd = endOfWeek(new Date());
    
    const thisWeekSessions = studySessions.filter(session => {
      const sessionDate = new Date(session.date);
      return sessionDate >= weekStart && sessionDate <= weekEnd;
    });

    const totalMinutes = thisWeekSessions.reduce((sum, session) => sum + session.duration_minutes, 0);
    const avgProductivity = thisWeekSessions.length > 0 
      ? thisWeekSessions.reduce((sum, session) => sum + session.productivity_rating, 0) / thisWeekSessions.length 
      : 0;

    return { totalMinutes, avgProductivity, sessionCount: thisWeekSessions.length };
  };

  const getUpcomingExams = () => {
    return exams
      .filter(exam => new Date(exam.exam_date) >= new Date())
      .sort((a, b) => new Date(a.exam_date) - new Date(b.exam_date))
      .slice(0, 3);
  };

  const getTodaysTasks = () => {
    return tasks.filter(task => 
      isToday(new Date(task.due_date)) && task.status !== 'completed'
    );
  };

  const getRecentMood = () => {
    return moodEntries[0] || null;
  };

  const { totalMinutes, avgProductivity, sessionCount } = getWeeklyStats();
  const upcomingExams = getUpcomingExams();
  const todaysTasks = getTodaysTasks();
  const recentMood = getRecentMood();

  if (isLoading) {
    return (
      <div className="p-6 space-y-6">
        <div className="animate-pulse">
          <div className="h-8 bg-slate-200 dark:bg-slate-700 rounded w-64 mb-4"></div>
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
            {Array(4).fill(0).map((_, i) => (
              <div key={i} className="h-32 bg-slate-200 dark:bg-slate-800 rounded-xl"></div>
            ))}
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="p-6 space-y-8 max-w-7xl mx-auto">
      {/* Header */}
      <div className="flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
        <div>
          <h1 className="text-3xl font-bold text-slate-900 dark:text-slate-50">Welcome back!</h1>
          <p className="text-slate-600 dark:text-slate-400 mt-1">Here's your study overview for today</p>
        </div>
        <Link to={createPageUrl("AIChat")}>
          <Button className="bg-gradient-to-r from-blue-500 to-indigo-600 hover:from-blue-600 hover:to-indigo-700 shadow-lg">
            <Brain className="w-4 h-4 mr-2" />
            Ask AI for Help
          </Button>
        </Link>
      </div>

      {/* Quick Stats */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <Card className="border-none shadow-lg bg-gradient-to-br from-blue-50 to-indigo-50 dark:from-slate-800 dark:to-slate-900 dark:shadow-none dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">This Week</CardTitle>
              <Clock className="w-5 h-5 text-blue-500" />
            </div>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">{Math.floor(totalMinutes / 60)}h {totalMinutes % 60}m</div>
            <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">{sessionCount} study sessions</p>
          </CardContent>
        </Card>

        <Card className="border-none shadow-lg bg-gradient-to-br from-green-50 to-emerald-50 dark:from-slate-800 dark:to-green-950/30 dark:shadow-none dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">Productivity</CardTitle>
              <TrendingUp className="w-5 h-5 text-green-500" />
            </div>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">{avgProductivity.toFixed(1)}/5</div>
            <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">Average this week</p>
          </CardContent>
        </Card>

        <Card className="border-none shadow-lg bg-gradient-to-br from-orange-50 to-red-50 dark:from-slate-800 dark:to-orange-950/30 dark:shadow-none dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">Next Exam</CardTitle>
              <Target className="w-5 h-5 text-orange-500" />
            </div>
          </CardHeader>
          <CardContent>
            {upcomingExams[0] ? (
              <>
                <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">
                  {differenceInDays(new Date(upcomingExams[0].exam_date), new Date())} days
                </div>
                <p className="text-sm text-slate-600 dark:text-slate-400 mt-1 truncate">{upcomingExams[0].subject}</p>
              </>
            ) : (
              <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">-</div>
            )}
          </CardContent>
        </Card>

        <Card className="border-none shadow-lg bg-gradient-to-br from-purple-50 to-pink-50 dark:from-slate-800 dark:to-purple-950/30 dark:shadow-none dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">Today's Mood</CardTitle>
              <Heart className="w-5 h-5 text-purple-500" />
            </div>
          </CardHeader>
          <CardContent>
            {recentMood ? (
              <>
                <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">{recentMood.mood_score}/10</div>
                <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">Feeling good!</p>
              </>
            ) : (
              <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">-</div>
            )}
          </CardContent>
        </Card>
      </div>

      {/* Main Content Grid */}
      <div className="grid lg:grid-cols-3 gap-8">
        {/* Left Column */}
        <div className="lg:col-span-2 space-y-6">
          {/* Today's Tasks */}
          <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:shadow-none dark:border dark:border-slate-700/60">
            <CardHeader>
              <div className="flex items-center justify-between">
                <CardTitle className="flex items-center gap-2 text-slate-900 dark:text-slate-100">
                  <Calendar className="w-5 h-5 text-blue-500" />
                  Today's Tasks
                </CardTitle>
                <Link to={createPageUrl("Planner")}>
                  <Button variant="outline" size="sm" className="bg-transparent dark:bg-slate-700/50 dark:text-slate-300 dark:border-slate-600 dark:hover:bg-slate-700">
                    View All <ArrowRight className="w-4 h-4 ml-1" />
                  </Button>
                </Link>
              </div>
            </CardHeader>
            <CardContent>
              {todaysTasks.length > 0 ? (
                <div className="space-y-3">
                  {todaysTasks.slice(0, 3).map(task => (
                    <div key={task.id} className="flex items-center gap-3 p-3 bg-slate-50 dark:bg-slate-700/50 rounded-lg">
                      <div className={`w-3 h-3 rounded-full ${
                        task.priority === 'urgent' ? 'bg-red-500' :
                        task.priority === 'high' ? 'bg-orange-500' :
                        task.priority === 'medium' ? 'bg-yellow-500' : 'bg-green-500'
                      }`} />
                      <div className="flex-1">
                        <p className="font-medium text-slate-900 dark:text-slate-100">{task.title}</p>
                        <p className="text-sm text-slate-600 dark:text-slate-400">{task.subject}</p>
                      </div>
                      <span className="text-xs bg-slate-200 dark:bg-slate-600 dark:text-slate-300 px-2 py-1 rounded-full">
                        {task.estimated_duration}min
                      </span>
                    </div>
                  ))}
                </div>
              ) : (
                <p className="text-slate-600 dark:text-slate-400 text-center py-8">No tasks due today! 🎉</p>
              )}
            </CardContent>
          </Card>

          {/* Quick Actions */}
          <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:shadow-none dark:border dark:border-slate-700/60">
            <CardHeader>
              <CardTitle className="text-slate-900 dark:text-slate-100">Quick Actions</CardTitle>
            </CardHeader>
            <CardContent>
              <div className="grid grid-cols-2 gap-4">
                <Link to={createPageUrl("Planner")}>
                  <Button variant="outline" className="w-full h-16 flex-col bg-transparent text-slate-700 dark:bg-slate-700/50 dark:text-slate-300 dark:border-slate-600 dark:hover:bg-slate-700">
                    <Plus className="w-5 h-5 mb-1" />
                    Add Task
                  </Button>
                </Link>
                <Link to={createPageUrl("Flashcards")}>
                  <Button variant="outline" className="w-full h-16 flex-col bg-transparent text-slate-700 dark:bg-slate-700/50 dark:text-slate-300 dark:border-slate-600 dark:hover:bg-slate-700">
                    <BookOpen className="w-5 h-5 mb-1" />
                    Study Cards
                  </Button>
                </Link>
                <Link to={createPageUrl("StudyGroups")}>
                  <Button variant="outline" className="w-full h-16 flex-col bg-transparent text-slate-700 dark:bg-slate-700/50 dark:text-slate-300 dark:border-slate-600 dark:hover:bg-slate-700">
                    <Users className="w-5 h-5 mb-1" />
                    Find Group
                  </Button>
                </Link>
                <Link to={createPageUrl("MoodTracker")}>
                  <Button variant="outline" className="w-full h-16 flex-col bg-transparent text-slate-700 dark:bg-slate-700/50 dark:text-slate-300 dark:border-slate-600 dark:hover:bg-slate-700">
                    <Heart className="w-5 h-5 mb-1" />
                    Track Mood
                  </Button>
                </Link>
              </div>
            </CardContent>
          </Card>
        </div>

        {/* Right Column */}
        <div className="space-y-6">
          {/* Upcoming Exams */}
          <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:shadow-none dark:border dark:border-slate-700/60">
            <CardHeader>
              <div className="flex items-center justify-between">
                <CardTitle className="flex items-center gap-2 text-slate-900 dark:text-slate-100">
                  <Target className="w-5 h-5 text-orange-500" />
                  Upcoming Exams
                </CardTitle>
                <Link to={createPageUrl("Exams")}>
                  <Button variant="outline" size="sm" className="bg-transparent dark:bg-slate-700/50 dark:text-slate-300 dark:border-slate-600 dark:hover:bg-slate-700">View All</Button>
                </Link>
              </div>
            </CardHeader>
            <CardContent>
              {upcomingExams.length > 0 ? (
                <div className="space-y-3">
                  {upcomingExams.map(exam => (
                    <div key={exam.id} className="p-3 bg-orange-50 dark:bg-orange-900/20 rounded-lg border border-orange-100 dark:border-orange-800/50">
                      <div className="flex items-start justify-between">
                        <div>
                          <p className="font-medium text-slate-900 dark:text-slate-100">{exam.subject}</p>
                          <p className="text-sm text-slate-600 dark:text-slate-400">{exam.exam_name}</p>
                          <p className="text-xs text-orange-600 dark:text-orange-400 font-medium mt-1">
                            {format(new Date(exam.exam_date), "MMM d, yyyy")}
                          </p>
                        </div>
                        <span className="text-lg font-bold text-orange-600 dark:text-orange-400">
                          {differenceInDays(new Date(exam.exam_date), new Date())}d
                        </span>
                      </div>
                    </div>
                  ))}
                </div>
              ) : (
                <p className="text-slate-600 dark:text-slate-400 text-center py-4">No upcoming exams</p>
              )}
            </CardContent>
          </Card>

          {/* Study Stats */}
          <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:shadow-none dark:border dark:border-slate-700/60">
            <CardHeader>
              <CardTitle className="flex items-center gap-2 text-slate-900 dark:text-slate-100">
                <BarChart3 className="w-5 h-5 text-blue-500" />
                Weekly Progress
              </CardTitle>
            </CardHeader>
            <CardContent>
              <div className="space-y-4">
                <div className="flex justify-between items-center">
                  <span className="text-sm text-slate-600 dark:text-slate-400">Total Flashcards</span>
                  <span className="font-semibold text-slate-900 dark:text-slate-200">{flashcards.length}</span>
                </div>
                <div className="flex justify-between items-center">
                  <span className="text-sm text-slate-600 dark:text-slate-400">Completed Tasks</span>
                  <span className="font-semibold text-green-600 dark:text-green-400">{tasks.filter(t => t.status === 'completed').length}</span>
                </div>
                <div className="flex justify-between items-center">
                  <span className="text-sm text-slate-600 dark:text-slate-400">Study Sessions</span>
                  <span className="font-semibold text-slate-900 dark:text-slate-200">{sessionCount}</span>
                </div>
              </div>
            </CardContent>
          </Card>
        </div>
      </div>
    </div>
  );
}

import React, { useState, useRef, useEffect } from "react";
import { InvokeLLM } from "@/integrations/Core";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card } from "@/components/ui/card";
import { Send, Bot, User, Sparkles, BookOpen, Calculator, Lightbulb } from "lucide-react";
import { motion, AnimatePresence } from "framer-motion";

const STUDY_PROMPTS = [
  {
    icon: BookOpen,
    title: "Explain a concept",
    prompt: "Can you explain [topic] in simple terms with examples?"
  },
  {
    icon: Calculator,
    title: "Solve a problem",
    prompt: "Help me solve this step by step: [problem]"
  },
  {
    icon: Lightbulb,
    title: "Study tips",
    prompt: "What are the best study techniques for [subject]?"
  },
  {
    icon: Sparkles,
    title: "Quiz me",
    prompt: "Create a practice quiz on [topic] with 5 questions"
  }
];

export default function AIChat() {
  const [messages, setMessages] = useState([
    {
      type: 'assistant',
      content: "Hi! I'm your AI study assistant. I can help you understand concepts, solve problems, create study plans, and much more. What would you like to study today?",
      timestamp: new Date()
    }
  ]);
  const [input, setInput] = useState("");
  const [isLoading, setIsLoading] = useState(false);
  const messagesEndRef = useRef(null);
  const inputRef = useRef(null);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  };

  useEffect(() => {
    scrollToBottom();
  }, [messages]);

  const sendMessage = async (messageText) => {
    if (!messageText.trim() || isLoading) return;

    const userMessage = {
      type: 'user',
      content: messageText,
      timestamp: new Date()
    };

    setMessages(prev => [...prev, userMessage]);
    setInput("");
    setIsLoading(true);

    try {
      const response = await InvokeLLM({
        prompt: `You are an AI study assistant helping a student. Be helpful, encouraging, and educational. Break down complex topics into digestible parts. Use examples and analogies when helpful. 

Student question: ${messageText}

Please provide a comprehensive, helpful response that aids their learning.`,
        add_context_from_internet: true
      });

      const assistantMessage = {
        type: 'assistant',
        content: response,
        timestamp: new Date()
      };

      setMessages(prev => [...prev, assistantMessage]);
    } catch (error) {
      console.error("Error getting AI response:", error);
      const errorMessage = {
        type: 'assistant',
        content: "I'm sorry, I encountered an error. Please try asking your question again.",
        timestamp: new Date()
      };
      setMessages(prev => [...prev, errorMessage]);
    }

    setIsLoading(false);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    sendMessage(input);
  };

  const handlePromptClick = (prompt) => {
    setInput(prompt);
    inputRef.current?.focus();
  };

  return (
    <div className="flex flex-col h-screen bg-gradient-to-br from-slate-50 via-blue-50 to-indigo-50 dark:from-slate-900 dark:via-slate-900 dark:to-blue-950">
      {/* Header */}
      <div className="bg-white/90 dark:bg-slate-900/90 backdrop-blur-sm border-b border-slate-200/60 dark:border-slate-700/60 p-6 shadow-sm">
        <div className="max-w-4xl mx-auto">
          <div className="flex items-center gap-3">
            <div className="w-10 h-10 bg-gradient-to-br from-blue-500 to-indigo-600 rounded-xl flex items-center justify-center">
              <Bot className="w-5 h-5 text-white" />
            </div>
            <div>
              <h1 className="text-2xl font-bold text-slate-900 dark:text-slate-50">AI Study Assistant</h1>
              <p className="text-slate-600 dark:text-slate-400">Get instant help with your studies</p>
            </div>
          </div>
        </div>
      </div>

      {/* Messages */}
      <div className="flex-1 overflow-auto p-6">
        <div className="max-w-4xl mx-auto space-y-6">
          {/* Quick Prompts - Show only at the beginning */}
          {messages.length <= 1 && (
            <motion.div
              initial={{ opacity: 0, y: 20 }}
              animate={{ opacity: 1, y: 0 }}
              className="grid grid-cols-1 md:grid-cols-2 gap-4 mb-8"
            >
              {STUDY_PROMPTS.map((prompt, index) => (
                <Card
                  key={index}
                  className="p-4 cursor-pointer hover:shadow-lg dark:hover:bg-slate-700 transition-all duration-200 bg-white/80 dark:bg-slate-800/80 backdrop-blur-sm border-slate-200/60 dark:border-slate-700/60"
                  onClick={() => handlePromptClick(prompt.prompt)}
                >
                  <div className="flex items-center gap-3">
                    <div className="w-10 h-10 bg-gradient-to-br from-blue-100 to-indigo-100 dark:from-blue-900/50 dark:to-indigo-900/50 rounded-lg flex items-center justify-center">
                      <prompt.icon className="w-5 h-5 text-blue-600 dark:text-blue-400" />
                    </div>
                    <div>
                      <h3 className="font-semibold text-slate-900 dark:text-slate-100">{prompt.title}</h3>
                      <p className="text-sm text-slate-600 dark:text-slate-400">{prompt.prompt}</p>
                    </div>
                  </div>
                </Card>
              ))}
            </motion.div>
          )}

          {/* Chat Messages */}
          <AnimatePresence>
            {messages.map((message, index) => (
              <motion.div
                key={index}
                initial={{ opacity: 0, y: 20 }}
                animate={{ opacity: 1, y: 0 }}
                className={`flex gap-4 ${message.type === 'user' ? 'justify-end' : 'justify-start'}`}
              >
                {message.type === 'assistant' && (
                  <div className="w-8 h-8 bg-gradient-to-br from-blue-500 to-indigo-600 rounded-full flex items-center justify-center flex-shrink-0">
                    <Bot className="w-4 h-4 text-white" />
                  </div>
                )}
                
                <div className={`max-w-3xl ${message.type === 'user' ? 'order-first' : ''}`}>
                  <Card className={`p-4 ${
                    message.type === 'user' 
                      ? 'bg-gradient-to-r from-blue-500 to-indigo-600 text-white border-none' 
                      : 'bg-white/90 dark:bg-slate-800/90 backdrop-blur-sm border-slate-200/60 dark:border-slate-700/60 text-slate-800 dark:text-slate-200'
                  }`}>
                    <div className="whitespace-pre-wrap">{message.content}</div>
                    <div className={`text-xs mt-2 opacity-70 ${
                      message.type === 'user' ? 'text-blue-100' : 'text-slate-500 dark:text-slate-400'
                    }`}>
                      {message.timestamp.toLocaleTimeString()}
                    </div>
                  </Card>
                </div>

                {message.type === 'user' && (
                  <div className="w-8 h-8 bg-gradient-to-br from-slate-400 to-slate-600 dark:from-slate-600 dark:to-slate-700 rounded-full flex items-center justify-center flex-shrink-0">
                    <User className="w-4 h-4 text-white" />
                  </div>
                )}
              </motion.div>
            ))}
          </AnimatePresence>

          {/* Loading indicator */}
          {isLoading && (
            <motion.div
              initial={{ opacity: 0, y: 20 }}
              animate={{ opacity: 1, y: 0 }}
              className="flex gap-4 justify-start"
            >
              <div className="w-8 h-8 bg-gradient-to-br from-blue-500 to-indigo-600 rounded-full flex items-center justify-center">
                <Bot className="w-4 h-4 text-white" />
              </div>
              <Card className="p-4 bg-white/90 dark:bg-slate-800/90 backdrop-blur-sm border-slate-200/60 dark:border-slate-700/60">
                <div className="flex gap-1">
                  <div className="w-2 h-2 bg-slate-400 dark:bg-slate-500 rounded-full animate-bounce"></div>
                  <div className="w-2 h-2 bg-slate-400 dark:bg-slate-500 rounded-full animate-bounce" style={{ animationDelay: '0.1s' }}></div>
                  <div className="w-2 h-2 bg-slate-400 dark:bg-slate-500 rounded-full animate-bounce" style={{ animationDelay: '0.2s' }}></div>
                </div>
              </Card>
            </motion.div>
          )}

          <div ref={messagesEndRef} />
        </div>
      </div>

      {/* Input */}
      <div className="bg-white/90 dark:bg-slate-900/90 backdrop-blur-sm border-t border-slate-200/60 dark:border-slate-700/60 p-6">
        <div className="max-w-4xl mx-auto">
          <form onSubmit={handleSubmit} className="flex gap-4">
            <Input
              ref={inputRef}
              value={input}
              onChange={(e) => setInput(e.target.value)}
              placeholder="Ask me anything about your studies..."
              className="flex-1 bg-white/80 dark:bg-slate-800 border-slate-200/60 dark:border-slate-700 text-slate-900 dark:text-slate-100 placeholder:text-slate-500 dark:placeholder:text-slate-400 focus:ring-2 focus:ring-blue-500/20"
              disabled={isLoading}
            />
            <Button 
              type="submit" 
              disabled={isLoading || !input.trim()}
              className="bg-gradient-to-r from-blue-500 to-indigo-600 hover:from-blue-600 hover:to-indigo-700 shadow-lg"
            >
              <Send className="w-4 h-4" />
            </Button>
          </form>
        </div>
      </div>
    </div>
  );
}

import React, { useState, useEffect } from "react";
import { Task } from "@/entities/Task";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Calendar, Plus, Filter } from "lucide-react";
import { format, startOfWeek, endOfWeek, startOfMonth, endOfMonth, isToday, parseISO } from "date-fns";

import TaskForm from "../components/planner/TaskForm";
import TaskList from "../components/planner/TaskList";
import WeekView from "../components/planner/WeekView";
import MonthView from "../components/planner/MonthView";

export default function Planner() {
  const [tasks, setTasks] = useState([]);
  const [showTaskForm, setShowTaskForm] = useState(false);
  const [editingTask, setEditingTask] = useState(null);
  const [activeView, setActiveView] = useState("day");
  const [selectedDate, setSelectedDate] = useState(new Date());

  useEffect(() => {
    loadTasks();
  }, []);

  const loadTasks = async () => {
    const data = await Task.list("-due_date", 100);
    setTasks(data);
  };

  const handleTaskSubmit = async (taskData) => {
    if (editingTask) {
      await Task.update(editingTask.id, taskData);
    } else {
      await Task.create(taskData);
    }
    setShowTaskForm(false);
    setEditingTask(null);
    loadTasks();
  };

  const handleTaskUpdate = async (taskId, updates) => {
    await Task.update(taskId, updates);
    loadTasks();
  };

  const handleTaskDelete = async (taskId) => {
    await Task.delete(taskId);
    loadTasks();
  };

  const getFilteredTasks = () => {
    const now = new Date();
    switch (activeView) {
      case "day":
        return tasks.filter(task => isToday(parseISO(task.due_date)));
      case "week":
        const weekStart = startOfWeek(selectedDate);
        const weekEnd = endOfWeek(selectedDate);
        return tasks.filter(task => {
          const taskDate = parseISO(task.due_date);
          return taskDate >= weekStart && taskDate <= weekEnd;
        });
      case "month":
        const monthStart = startOfMonth(selectedDate);
        const monthEnd = endOfMonth(selectedDate);
        return tasks.filter(task => {
          const taskDate = parseISO(task.due_date);
          return taskDate >= monthStart && taskDate <= monthEnd;
        });
      default:
        return tasks;
    }
  };

  return (
    <div className="p-6 max-w-7xl mx-auto">
      <div className="flex flex-col md:flex-row justify-between items-start md:items-center mb-8 gap-4">
        <div>
          <h1 className="text-3xl font-bold text-slate-900 dark:text-slate-50">Study Planner</h1>
          <p className="text-slate-600 dark:text-slate-400 mt-1">Organize your tasks and assignments</p>
        </div>
        <Button
          onClick={() => setShowTaskForm(true)}
          className="bg-gradient-to-r from-blue-500 to-indigo-600 hover:from-blue-600 hover:to-indigo-700 shadow-lg"
        >
          <Plus className="w-4 h-4 mr-2" />
          Add Task
        </Button>
      </div>

      <div className="grid lg:grid-cols-4 gap-8">
        {/* Main Content */}
        <div className="lg:col-span-3">
          <Tabs value={activeView} onValueChange={setActiveView} className="w-full">
            <TabsList className="grid w-full grid-cols-3 mb-6 bg-slate-100 dark:bg-slate-800 text-slate-600 dark:text-slate-300">
              <TabsTrigger value="day" className="data-[state=active]:bg-white data-[state=active]:text-slate-900 data-[state=active]:shadow-md dark:data-[state=active]:bg-slate-700 dark:data-[state=active]:text-slate-100">Today</TabsTrigger>
              <TabsTrigger value="week" className="data-[state=active]:bg-white data-[state=active]:text-slate-900 data-[state=active]:shadow-md dark:data-[state=active]:bg-slate-700 dark:data-[state=active]:text-slate-100">This Week</TabsTrigger>
              <TabsTrigger value="month" className="data-[state=active]:bg-white data-[state=active]:text-slate-900 data-[state=active]:shadow-md dark:data-[state=active]:bg-slate-700 dark:data-[state=active]:text-slate-100">This Month</TabsTrigger>
            </TabsList>

            <TabsContent value="day">
              <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:shadow-none dark:border dark:border-slate-700/60">
                <CardHeader>
                  <CardTitle className="flex items-center gap-2 text-slate-900 dark:text-slate-100">
                    <Calendar className="w-5 h-5 text-blue-500" />
                    Today's Tasks - {format(new Date(), "MMMM d, yyyy")}
                  </CardTitle>
                </CardHeader>
                <CardContent>
                  <TaskList 
                    tasks={getFilteredTasks()}
                    onTaskUpdate={handleTaskUpdate}
                    onTaskEdit={setEditingTask}
                    onTaskDelete={handleTaskDelete}
                  />
                </CardContent>
              </Card>
            </TabsContent>

            <TabsContent value="week">
              <WeekView 
                tasks={getFilteredTasks()}
                selectedDate={selectedDate}
                onTaskUpdate={handleTaskUpdate}
                onTaskEdit={setEditingTask}
              />
            </TabsContent>

            <TabsContent value="month">
              <MonthView 
                tasks={getFilteredTasks()}
                selectedDate={selectedDate}
                onDateSelect={setSelectedDate}
                onTaskUpdate={handleTaskUpdate}
              />
            </TabsContent>
          </Tabs>
        </div>

        {/* Sidebar */}
        <div className="space-y-6">
          {/* Quick Stats */}
          <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:shadow-none dark:border dark:border-slate-700/60">
            <CardHeader>
              <CardTitle className="text-lg text-slate-900 dark:text-slate-100">Quick Stats</CardTitle>
            </CardHeader>
            <CardContent className="space-y-4">
              <div className="flex justify-between">
                <span className="text-slate-600 dark:text-slate-400">Total Tasks</span>
                <span className="font-semibold text-slate-900 dark:text-slate-200">{tasks.length}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-600 dark:text-slate-400">Completed</span>
                <span className="font-semibold text-green-600 dark:text-green-400">
                  {tasks.filter(t => t.status === 'completed').length}
                </span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-600 dark:text-slate-400">In Progress</span>
                <span className="font-semibold text-blue-600 dark:text-blue-400">
                  {tasks.filter(t => t.status === 'in_progress').length}
                </span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-600 dark:text-slate-400">Overdue</span>
                <span className="font-semibold text-red-600 dark:text-red-400">
                  {tasks.filter(t => t.status !== 'completed' && new Date(t.due_date) < new Date()).length}
                </span>
              </div>
            </CardContent>
          </Card>

          {/* Subject Breakdown */}
          <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:shadow-none dark:border dark:border-slate-700/60">
            <CardHeader>
              <CardTitle className="text-lg text-slate-900 dark:text-slate-100">By Subject</CardTitle>
            </CardHeader>
            <CardContent>
              {Object.entries(
                tasks.reduce((acc, task) => {
                  const subject = task.subject || 'Other';
                  acc[subject] = (acc[subject] || 0) + 1;
                  return acc;
                }, {})
              ).map(([subject, count]) => (
                <div key={subject} className="flex justify-between items-center py-2">
                  <span className="text-slate-600 dark:text-slate-400">{subject}</span>
                  <span className="font-semibold text-slate-900 dark:text-slate-200">{count}</span>
                </div>
              ))}
            </CardContent>
          </Card>
        </div>
      </div>

      {/* Task Form Modal */}
      {(showTaskForm || editingTask) && (
        <div className="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center p-4 z-50">
          <div className="bg-white dark:bg-slate-800 rounded-2xl shadow-2xl max-w-md w-full max-h-[90vh] overflow-y-auto">
            <TaskForm
              task={editingTask}
              onSubmit={handleTaskSubmit}
              onCancel={() => {
                setShowTaskForm(false);
                setEditingTask(null);
              }}
            />
          </div>
        </div>
      )}
    </div>
  );
}
import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Badge } from "@/components/ui/badge";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { 
  Plus, 
  Edit, 
  Trash2, 
  RotateCw, 
  Check, 
  X, 
  ChevronRight,
  BookOpen,
  StickyNote
} from "lucide-react";
import { motion, AnimatePresence } from "framer-motion";

export default function Flashcards() {
  const [activeTab, setActiveTab] = useState("decks");
  const [showCreateForm, setShowCreateForm] = useState(false);
  const [selectedDeck, setSelectedDeck] = useState(null);
  const [studyMode, setStudyMode] = useState(false);
  const [currentCardIndex, setCurrentCardIndex] = useState(0);
  const [isFlipped, setIsFlipped] = useState(false);
  const [notepad, setNotepad] = useState("");
  const [showNotepad, setShowNotepad] = useState(false);
  
  const [formData, setFormData] = useState({
    front: "",
    back: "",
    subject: "",
    deck_name: "",
    difficulty: "medium"
  });

  const queryClient = useQueryClient();

  const { data: flashcards = [], isLoading } = useQuery({
    queryKey: ['flashcards'],
    queryFn: () => base44.entities.Flashcard.list("-created_date", 200),
  });

  const createMutation = useMutation({
    mutationFn: (data) => base44.entities.Flashcard.create(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['flashcards'] });
      setShowCreateForm(false);
      setFormData({ front: "", back: "", subject: "", deck_name: "", difficulty: "medium" });
    },
  });

  const deleteMutation = useMutation({
    mutationFn: (id) => base44.entities.Flashcard.delete(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['flashcards'] });
    },
  });

  const updateMutation = useMutation({
    mutationFn: ({ id, data }) => base44.entities.Flashcard.update(id, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['flashcards'] });
    },
  });

  const getDecks = () => {
    const deckMap = {};
    flashcards.forEach(card => {
      const deckName = card.deck_name || "Uncategorized";
      if (!deckMap[deckName]) {
        deckMap[deckName] = {
          name: deckName,
          subject: card.subject,
          cards: []
        };
      }
      deckMap[deckName].cards.push(card);
    });
    return Object.values(deckMap);
  };

  const handleCreateCard = () => {
    createMutation.mutate(formData);
  };

  const handleStartStudy = (deck) => {
    setSelectedDeck(deck);
    setCurrentCardIndex(0);
    setIsFlipped(false);
    setStudyMode(true);
    setShowNotepad(false);
    setNotepad("");
  };

  const handleNextCard = () => {
    setIsFlipped(false);
    if (currentCardIndex < selectedDeck.cards.length - 1) {
      setCurrentCardIndex(currentCardIndex + 1);
    } else {
      setStudyMode(false);
      setSelectedDeck(null);
    }
  };

  const handleMarkCard = async (success) => {
    const card = selectedDeck.cards[currentCardIndex];
    await updateMutation.mutateAsync({
      id: card.id,
      data: {
        review_count: (card.review_count || 0) + 1,
        success_count: (card.success_count || 0) + (success ? 1 : 0)
      }
    });
    handleNextCard();
  };

  const decks = getDecks();
  const currentCard = studyMode && selectedDeck ? selectedDeck.cards[currentCardIndex] : null;

  if (studyMode && currentCard) {
    return (
      <div className="min-h-screen bg-gradient-to-br from-slate-50 via-blue-50 to-indigo-50 dark:from-slate-900 dark:via-slate-900 dark:to-blue-950 p-6">
        <div className="max-w-4xl mx-auto">
          <div className="flex justify-between items-center mb-8">
            <div>
              <h1 className="text-3xl font-bold text-slate-900 dark:text-slate-50">{selectedDeck.name}</h1>
              <p className="text-slate-600 dark:text-slate-400">
                Card {currentCardIndex + 1} of {selectedDeck.cards.length}
              </p>
            </div>
            <Button 
              variant="outline" 
              onClick={() => setStudyMode(false)}
              className="dark:bg-slate-800 dark:text-slate-300 dark:border-slate-700"
            >
              Exit Study Mode
            </Button>
          </div>

          <div className="grid lg:grid-cols-3 gap-6">
            <div className="lg:col-span-2">
              {/* Flashcard */}
              <motion.div
                key={currentCardIndex}
                initial={{ opacity: 0, x: 50 }}
                animate={{ opacity: 1, x: 0 }}
                className="mb-6"
              >
                <Card 
                  className="h-96 cursor-pointer border-none shadow-2xl bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60"
                  onClick={() => setIsFlipped(!isFlipped)}
                >
                  <CardContent className="flex items-center justify-center h-full p-8">
                    <AnimatePresence mode="wait">
                      <motion.div
                        key={isFlipped ? 'back' : 'front'}
                        initial={{ rotateY: 90, opacity: 0 }}
                        animate={{ rotateY: 0, opacity: 1 }}
                        exit={{ rotateY: -90, opacity: 0 }}
                        transition={{ duration: 0.3 }}
                        className="text-center w-full"
                      >
                        <div className="text-sm text-slate-500 dark:text-slate-400 mb-4">
                          {isFlipped ? 'Answer' : 'Question'}
                        </div>
                        <div className="text-2xl font-semibold text-slate-900 dark:text-slate-100">
                          {isFlipped ? currentCard.back : currentCard.front}
                        </div>
                        <div className="mt-8 text-slate-500 dark:text-slate-400">
                          Click to flip
                        </div>
                      </motion.div>
                    </AnimatePresence>
                  </CardContent>
                </Card>
              </motion.div>

              {/* Action Buttons */}
              <div className="flex gap-4 justify-center">
                <Button
                  size="lg"
                  variant="outline"
                  onClick={() => handleMarkCard(false)}
                  className="bg-red-50 dark:bg-red-950/30 text-red-600 dark:text-red-400 border-red-200 dark:border-red-800 hover:bg-red-100 dark:hover:bg-red-950/50"
                  disabled={!isFlipped}
                >
                  <X className="w-5 h-5 mr-2" />
                  Need Practice
                </Button>
                <Button
                  size="lg"
                  onClick={() => handleMarkCard(true)}
                  className="bg-gradient-to-r from-green-500 to-emerald-600 hover:from-green-600 hover:to-emerald-700"
                  disabled={!isFlipped}
                >
                  <Check className="w-5 h-5 mr-2" />
                  Got It!
                </Button>
              </div>
            </div>

            {/* Orange Notepad */}
            <div className="space-y-4">
              <Button
                onClick={() => setShowNotepad(!showNotepad)}
                className="w-full bg-gradient-to-r from-orange-500 to-orange-600 hover:from-orange-600 hover:to-orange-700 shadow-lg"
              >
                <StickyNote className="w-4 h-4 mr-2" />
                {showNotepad ? 'Hide Notepad' : 'Show Notepad'}
              </Button>

              <AnimatePresence>
                {showNotepad && (
                  <motion.div
                    initial={{ opacity: 0, y: -20 }}
                    animate={{ opacity: 1, y: 0 }}
                    exit={{ opacity: 0, y: -20 }}
                  >
                    <Card className="border-4 border-orange-500 dark:border-orange-600 shadow-xl bg-gradient-to-br from-orange-50 to-orange-100 dark:from-orange-950/40 dark:to-orange-900/30">
                      <CardHeader className="bg-gradient-to-r from-orange-500 to-orange-600 text-white">
                        <CardTitle className="flex items-center gap-2 text-lg">
                          <StickyNote className="w-5 h-5" />
                          Quick Notes
                        </CardTitle>
                      </CardHeader>
                      <CardContent className="p-4">
                        <Textarea
                          value={notepad}
                          onChange={(e) => setNotepad(e.target.value)}
                          placeholder="Write your notes here..."
                          className="min-h-[300px] bg-white dark:bg-slate-800/80 border-orange-300 dark:border-orange-700/50 focus:border-orange-500 dark:focus:border-orange-600 text-slate-900 dark:text-slate-100 placeholder:text-slate-500 dark:placeholder:text-slate-400"
                        />
                        <p className="text-xs text-orange-700 dark:text-orange-400 mt-2 font-medium">
                          💡 These notes are temporary and will be cleared when you exit study mode
                        </p>
                      </CardContent>
                    </Card>
                  </motion.div>
                )}
              </AnimatePresence>

              <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60">
                <CardHeader>
                  <CardTitle className="text-lg text-slate-900 dark:text-slate-100">Card Info</CardTitle>
                </CardHeader>
                <CardContent className="space-y-3">
                  <div className="flex justify-between">
                    <span className="text-slate-600 dark:text-slate-400">Subject</span>
                    <span className="font-semibold text-slate-900 dark:text-slate-200">{currentCard.subject}</span>
                  </div>
                  <div className="flex justify-between">
                    <span className="text-slate-600 dark:text-slate-400">Difficulty</span>
                    <Badge className={
                      currentCard.difficulty === 'hard' ? 'bg-red-100 text-red-800 dark:bg-red-950/50 dark:text-red-400' :
                      currentCard.difficulty === 'medium' ? 'bg-yellow-100 text-yellow-800 dark:bg-yellow-950/50 dark:text-yellow-400' :
                      'bg-green-100 text-green-800 dark:bg-green-950/50 dark:text-green-400'
                    }>
                      {currentCard.difficulty}
                    </Badge>
                  </div>
                  <div className="flex justify-between">
                    <span className="text-slate-600 dark:text-slate-400">Reviews</span>
                    <span className="font-semibold text-slate-900 dark:text-slate-200">{currentCard.review_count || 0}</span>
                  </div>
                  <div className="flex justify-between">
                    <span className="text-slate-600 dark:text-slate-400">Success Rate</span>
                    <span className="font-semibold text-slate-900 dark:text-slate-200">
                      {currentCard.review_count > 0 
                        ? Math.round((currentCard.success_count / currentCard.review_count) * 100) 
                        : 0}%
                    </span>
                  </div>
                </CardContent>
              </Card>
            </div>
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="p-6 max-w-7xl mx-auto">
      <div className="flex flex-col md:flex-row justify-between items-start md:items-center mb-8 gap-4">
        <div>
          <h1 className="text-3xl font-bold text-slate-900 dark:text-slate-50">Flashcards</h1>
          <p className="text-slate-600 dark:text-slate-400 mt-1">Study smart with spaced repetition</p>
        </div>
        <Button
          onClick={() => setShowCreateForm(true)}
          className="bg-gradient-to-r from-blue-500 to-indigo-600 hover:from-blue-600 hover:to-indigo-700 shadow-lg"
        >
          <Plus className="w-4 h-4 mr-2" />
          Create Flashcard
        </Button>
      </div>

      <Tabs value={activeTab} onValueChange={setActiveTab} className="w-full">
        <TabsList className="mb-6 bg-slate-100 dark:bg-slate-800">
          <TabsTrigger value="decks" className="data-[state=active]:bg-white dark:data-[state=active]:bg-slate-700">Decks</TabsTrigger>
          <TabsTrigger value="all" className="data-[state=active]:bg-white dark:data-[state=active]:bg-slate-700">All Cards</TabsTrigger>
        </TabsList>

        <TabsContent value="decks">
          <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
            {decks.map((deck, index) => (
              <Card 
                key={index} 
                className="border-none shadow-lg hover:shadow-xl transition-all duration-200 bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60"
              >
                <CardHeader>
                  <CardTitle className="text-slate-900 dark:text-slate-100">{deck.name}</CardTitle>
                  <p className="text-sm text-slate-600 dark:text-slate-400">{deck.subject}</p>
                </CardHeader>
                <CardContent>
                  <div className="space-y-4">
                    <div className="flex justify-between items-center">
                      <span className="text-slate-600 dark:text-slate-400">Cards</span>
                      <span className="font-bold text-2xl text-blue-600 dark:text-blue-400">{deck.cards.length}</span>
                    </div>
                    <Button
                      onClick={() => handleStartStudy(deck)}
                      className="w-full bg-gradient-to-r from-blue-500 to-indigo-600 hover:from-blue-600 hover:to-indigo-700"
                    >
                      <BookOpen className="w-4 h-4 mr-2" />
                      Start Studying
                    </Button>
                  </div>
                </CardContent>
              </Card>
            ))}
          </div>
        </TabsContent>

        <TabsContent value="all">
          <div className="space-y-4">
            {flashcards.map(card => (
              <Card key={card.id} className="border-none shadow-lg bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60">
                <CardContent className="p-6">
                  <div className="flex justify-between items-start gap-4">
                    <div className="flex-1">
                      <div className="flex items-center gap-2 mb-2">
                        <Badge className="bg-blue-100 text-blue-800 dark:bg-blue-950/50 dark:text-blue-400">
                          {card.subject}
                        </Badge>
                        <Badge className="bg-slate-100 text-slate-800 dark:bg-slate-700 dark:text-slate-300">
                          {card.deck_name}
                        </Badge>
                      </div>
                      <div className="grid md:grid-cols-2 gap-4">
                        <div>
                          <p className="text-sm text-slate-600 dark:text-slate-400 mb-1">Question</p>
                          <p className="font-medium text-slate-900 dark:text-slate-100">{card.front}</p>
                        </div>
                        <div>
                          <p className="text-sm text-slate-600 dark:text-slate-400 mb-1">Answer</p>
                          <p className="font-medium text-slate-900 dark:text-slate-100">{card.back}</p>
                        </div>
                      </div>
                    </div>
                    <Button
                      variant="ghost"
                      size="icon"
                      onClick={() => deleteMutation.mutate(card.id)}
                      className="text-red-500 hover:text-red-700 dark:text-red-400 dark:hover:text-red-300"
                    >
                      <Trash2 className="w-4 h-4" />
                    </Button>
                  </div>
                </CardContent>
              </Card>
            ))}
          </div>
        </TabsContent>
      </Tabs>

      {/* Create Form Modal */}
      {showCreateForm && (
        <div className="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center p-4 z-50">
          <Card className="max-w-md w-full bg-white dark:bg-slate-800">
            <CardHeader>
              <CardTitle className="text-slate-900 dark:text-slate-100">Create Flashcard</CardTitle>
            </CardHeader>
            <CardContent className="space-y-4">
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Question (Front)</label>
                <Textarea
                  value={formData.front}
                  onChange={(e) => setFormData({...formData, front: e.target.value})}
                  placeholder="What is the question?"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Answer (Back)</label>
                <Textarea
                  value={formData.back}
                  onChange={(e) => setFormData({...formData, back: e.target.value})}
                  placeholder="What is the answer?"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Subject</label>
                <Input
                  value={formData.subject}
                  onChange={(e) => setFormData({...formData, subject: e.target.value})}
                  placeholder="e.g., Physics"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Deck Name</label>
                <Input
                  value={formData.deck_name}
                  onChange={(e) => setFormData({...formData, deck_name: e.target.value})}
                  placeholder="e.g., Physics Formulas"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Difficulty</label>
                <Select value={formData.difficulty} onValueChange={(value) => setFormData({...formData, difficulty: value})}>
                  <SelectTrigger className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700">
                    <SelectValue />
                  </SelectTrigger>
                  <SelectContent className="bg-white dark:bg-slate-800">
                    <SelectItem value="easy">Easy</SelectItem>
                    <SelectItem value="medium">Medium</SelectItem>
                    <SelectItem value="hard">Hard</SelectItem>
                  </SelectContent>
                </Select>
              </div>
              <div className="flex gap-3 pt-4">
                <Button variant="outline" onClick={() => setShowCreateForm(false)} className="flex-1 dark:bg-slate-700 dark:text-slate-300 dark:border-slate-600">
                  Cancel
                </Button>
                <Button onClick={handleCreateCard} className="flex-1 bg-gradient-to-r from-blue-500 to-indigo-600">
                  Create
                </Button>
              </div>
            </CardContent>
          </Card>
        </div>
      )}
    </div>
  );
}import React, { useState } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Badge } from "@/components/ui/badge";
import { Users, MapPin, Calendar, Plus, Edit, Trash2 } from "lucide-react";

export default function StudyGroups() {
  const [showCreateForm, setShowCreateForm] = useState(false);
  const [formData, setFormData] = useState({
    group_name: "",
    subject: "",
    description: "",
    meeting_schedule: "",
    meeting_location: "",
    contact_info: ""
  });

  const queryClient = useQueryClient();

  const { data: groups = [], isLoading } = useQuery({
    queryKey: ['studygroups'],
    queryFn: () => base44.entities.StudyGroup.list("-created_date", 50),
  });

  const createMutation = useMutation({
    mutationFn: (data) => base44.entities.StudyGroup.create(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['studygroups'] });
      setShowCreateForm(false);
      setFormData({ group_name: "", subject: "", description: "", meeting_schedule: "", meeting_location: "", contact_info: "" });
    },
  });

  const deleteMutation = useMutation({
    mutationFn: (id) => base44.entities.StudyGroup.delete(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['studygroups'] });
    },
  });

  const handleCreateGroup = () => {
    createMutation.mutate(formData);
  };

  return (
    <div className="p-6 max-w-7xl mx-auto">
      <div className="flex flex-col md:flex-row justify-between items-start md:items-center mb-8 gap-4">
        <div>
          <h1 className="text-3xl font-bold text-slate-900 dark:text-slate-50">Study Groups</h1>
          <p className="text-slate-600 dark:text-slate-400 mt-1">Find or create study groups to collaborate with peers</p>
        </div>
        <Button
          onClick={() => setShowCreateForm(true)}
          className="bg-gradient-to-r from-blue-500 to-indigo-600 hover:from-blue-600 hover:to-indigo-700 shadow-lg"
        >
          <Plus className="w-4 h-4 mr-2" />
          Create Study Group
        </Button>
      </div>

      {isLoading ? (
        <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
          {Array(6).fill(0).map((_, i) => (
            <div key={i} className="h-64 bg-slate-200 dark:bg-slate-800 rounded-xl animate-pulse"></div>
          ))}
        </div>
      ) : (
        <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
          {groups.map(group => (
            <Card key={group.id} className="border-none shadow-lg hover:shadow-xl transition-all duration-200 bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60">
              <CardHeader>
                <div className="flex justify-between items-start">
                  <div className="flex-1">
                    <CardTitle className="text-slate-900 dark:text-slate-100">{group.group_name}</CardTitle>
                    <Badge className="mt-2 bg-blue-100 text-blue-800 dark:bg-blue-950/50 dark:text-blue-400">
                      {group.subject}
                    </Badge>
                  </div>
                  <Button
                    variant="ghost"
                    size="icon"
                    onClick={() => deleteMutation.mutate(group.id)}
                    className="text-red-500 hover:text-red-700 dark:text-red-400"
                  >
                    <Trash2 className="w-4 h-4" />
                  </Button>
                </div>
              </CardHeader>
              <CardContent>
                <p className="text-slate-600 dark:text-slate-400 mb-4">{group.description}</p>
                
                <div className="space-y-3">
                  {group.meeting_schedule && (
                    <div className="flex items-center gap-2 text-sm text-slate-600 dark:text-slate-400">
                      <Calendar className="w-4 h-4 text-blue-500" />
                      <span>{group.meeting_schedule}</span>
                    </div>
                  )}
                  
                  {group.meeting_location && (
                    <div className="flex items-center gap-2 text-sm text-slate-600 dark:text-slate-400">
                      <MapPin className="w-4 h-4 text-green-500" />
                      <span>{group.meeting_location}</span>
                    </div>
                  )}
                  
                  <div className="flex items-center gap-2 text-sm text-slate-600 dark:text-slate-400">
                    <Users className="w-4 h-4 text-purple-500" />
                    <span>{group.member_count} member{group.member_count !== 1 ? 's' : ''}</span>
                  </div>

                  {group.contact_info && (
                    <div className="pt-3 border-t border-slate-200 dark:border-slate-700">
                      <p className="text-xs text-slate-500 dark:text-slate-400">Contact</p>
                      <p className="text-sm text-slate-900 dark:text-slate-200 font-medium">{group.contact_info}</p>
                    </div>
                  )}
                </div>

                <Button className="w-full mt-4 bg-gradient-to-r from-blue-500 to-indigo-600 hover:from-blue-600 hover:to-indigo-700">
                  Join Group
                </Button>
              </CardContent>
            </Card>
          ))}
        </div>
      )}

      {groups.length === 0 && !isLoading && (
        <div className="text-center py-16">
          <Users className="w-16 h-16 text-slate-300 dark:text-slate-700 mx-auto mb-4" />
          <h3 className="text-xl font-semibold text-slate-900 dark:text-slate-100 mb-2">No study groups yet</h3>
          <p className="text-slate-600 dark:text-slate-400">Be the first to create a study group!</p>
        </div>
      )}

      {/* Create Form Modal */}
      {showCreateForm && (
        <div className="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center p-4 z-50">
          <Card className="max-w-md w-full bg-white dark:bg-slate-800">
            <CardHeader>
              <CardTitle className="text-slate-900 dark:text-slate-100">Create Study Group</CardTitle>
            </CardHeader>
            <CardContent className="space-y-4">
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Group Name</label>
                <Input
                  value={formData.group_name}
                  onChange={(e) => setFormData({...formData, group_name: e.target.value})}
                  placeholder="e.g., Physics Study Group"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Subject</label>
                <Input
                  value={formData.subject}
                  onChange={(e) => setFormData({...formData, subject: e.target.value})}
                  placeholder="e.g., Physics"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Description</label>
                <Textarea
                  value={formData.description}
                  onChange={(e) => setFormData({...formData, description: e.target.value})}
                  placeholder="What will this group study?"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Meeting Schedule</label>
                <Input
                  value={formData.meeting_schedule}
                  onChange={(e) => setFormData({...formData, meeting_schedule: e.target.value})}
                  placeholder="e.g., Tuesdays 4-6 PM"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Location</label>
                <Input
                  value={formData.meeting_location}
                  onChange={(e) => setFormData({...formData, meeting_location: e.target.value})}
                  placeholder="e.g., Library Room 204"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Contact Info</label>
                <Input
                  value={formData.contact_info}
                  onChange={(e) => setFormData({...formData, contact_info: e.target.value})}
                  placeholder="Email or phone"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div className="flex gap-3 pt-4">
                <Button variant="outline" onClick={() => setShowCreateForm(false)} className="flex-1 dark:bg-slate-700 dark:text-slate-300 dark:border-slate-600">
                  Cancel
                </Button>
                <Button onClick={handleCreateGroup} className="flex-1 bg-gradient-to-r from-blue-500 to-indigo-600">
                  Create Group
                </Button>
              </div>
            </CardContent>
          </Card>
        </div>
      )}
    </div>
  );
}import React, { useState, useEffect } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery } from "@tanstack/react-query";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { 
  TrendingUp, 
  Clock, 
  Target, 
  BookOpen,
  Calendar,
  Award
} from "lucide-react";
import { 
  LineChart, 
  Line, 
  BarChart, 
  Bar, 
  PieChart, 
  Pie, 
  Cell,
  XAxis, 
  YAxis, 
  CartesianGrid, 
  Tooltip, 
  Legend, 
  ResponsiveContainer 
} from "recharts";
import { format, subDays, startOfWeek, endOfWeek } from "date-fns";

const COLORS = ['#3b82f6', '#10b981', '#f59e0b', '#ef4444', '#8b5cf6', '#ec4899'];

export default function Progress() {
  const { data: studySessions = [] } = useQuery({
    queryKey: ['studysessions'],
    queryFn: () => base44.entities.StudySession.list("-date", 100),
  });

  const { data: tasks = [] } = useQuery({
    queryKey: ['tasks'],
    queryFn: () => base44.entities.Task.list("-created_date", 100),
  });

  const { data: flashcards = [] } = useQuery({
    queryKey: ['flashcards'],
    queryFn: () => base44.entities.Flashcard.list("-created_date", 200),
  });

  const getLast7DaysData = () => {
    const data = [];
    for (let i = 6; i >= 0; i--) {
      const date = subDays(new Date(), i);
      const dateStr = format(date, 'yyyy-MM-dd');
      const sessions = studySessions.filter(s => s.date === dateStr);
      const totalMinutes = sessions.reduce((sum, s) => sum + s.duration_minutes, 0);
      
      data.push({
        date: format(date, 'MMM dd'),
        hours: Math.round(totalMinutes / 60 * 10) / 10,
        sessions: sessions.length
      });
    }
    return data;
  };

  const getSubjectDistribution = () => {
    const subjects = {};
    studySessions.forEach(session => {
      subjects[session.subject] = (subjects[session.subject] || 0) + session.duration_minutes;
    });
    return Object.entries(subjects).map(([name, value]) => ({
      name,
      value: Math.round(value / 60 * 10) / 10
    }));
  };

  const getProductivityTrend = () => {
    return studySessions
      .slice(0, 14)
      .reverse()
      .map(session => ({
        date: format(new Date(session.date), 'MMM dd'),
        productivity: session.productivity_rating
      }));
  };

  const getStats = () => {
    const totalMinutes = studySessions.reduce((sum, s) => sum + s.duration_minutes, 0);
    const avgProductivity = studySessions.length > 0
      ? studySessions.reduce((sum, s) => sum + s.productivity_rating, 0) / studySessions.length
      : 0;
    const completedTasks = tasks.filter(t => t.status === 'completed').length;
    const flashcardsMastered = flashcards.filter(f => 
      f.review_count > 3 && (f.success_count / f.review_count) > 0.8
    ).length;

    return { totalMinutes, avgProductivity, completedTasks, flashcardsMastered };
  };

  const stats = getStats();
  const weeklyData = getLast7DaysData();
  const subjectData = getSubjectDistribution();
  const productivityData = getProductivityTrend();

  return (
    <div className="p-6 max-w-7xl mx-auto">
      <div className="mb-8">
        <h1 className="text-3xl font-bold text-slate-900 dark:text-slate-50">Your Progress</h1>
        <p className="text-slate-600 dark:text-slate-400 mt-1">Track your learning journey and achievements</p>
      </div>

      {/* Stats Overview */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
        <Card className="border-none shadow-lg bg-gradient-to-br from-blue-50 to-indigo-50 dark:from-slate-800 dark:to-slate-900 dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">Total Study Time</CardTitle>
              <Clock className="w-5 h-5 text-blue-500" />
            </div>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">
              {Math.floor(stats.totalMinutes / 60)}h {stats.totalMinutes % 60}m
            </div>
            <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">All time</p>
          </CardContent>
        </Card>

        <Card className="border-none shadow-lg bg-gradient-to-br from-green-50 to-emerald-50 dark:from-slate-800 dark:to-green-950/30 dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">Avg Productivity</CardTitle>
              <TrendingUp className="w-5 h-5 text-green-500" />
            </div>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">{stats.avgProductivity.toFixed(1)}/5</div>
            <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">Overall rating</p>
          </CardContent>
        </Card>

        <Card className="border-none shadow-lg bg-gradient-to-br from-orange-50 to-red-50 dark:from-slate-800 dark:to-orange-950/30 dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">Tasks Completed</CardTitle>
              <Target className="w-5 h-5 text-orange-500" />
            </div>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">{stats.completedTasks}</div>
            <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">Total tasks done</p>
          </CardContent>
        </Card>

        <Card className="border-none shadow-lg bg-gradient-to-br from-purple-50 to-pink-50 dark:from-slate-800 dark:to-purple-950/30 dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">Cards Mastered</CardTitle>
              <Award className="w-5 h-5 text-purple-500" />
            </div>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">{stats.flashcardsMastered}</div>
            <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">80%+ success rate</p>
          </CardContent>
        </Card>
      </div>

      {/* Charts */}
      <Tabs defaultValue="weekly" className="w-full">
        <TabsList className="mb-6 bg-slate-100 dark:bg-slate-800">
          <TabsTrigger value="weekly" className="data-[state=active]:bg-white dark:data-[state=active]:bg-slate-700">Weekly Activity</TabsTrigger>
          <TabsTrigger value="subjects" className="data-[state=active]:bg-white dark:data-[state=active]:bg-slate-700">By Subject</TabsTrigger>
          <TabsTrigger value="productivity" className="data-[state=active]:bg-white dark:data-[state=active]:bg-slate-700">Productivity</TabsTrigger>
        </TabsList>

        <TabsContent value="weekly">
          <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60">
            <CardHeader>
              <CardTitle className="flex items-center gap-2 text-slate-900 dark:text-slate-100">
                <Calendar className="w-5 h-5 text-blue-500" />
                Last 7 Days Study Hours
              </CardTitle>
            </CardHeader>
            <CardContent>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={weeklyData}>
                  <CartesianGrid strokeDasharray="3 3" stroke="#e2e8f0" />
                  <XAxis dataKey="date" stroke="#64748b" />
                  <YAxis stroke="#64748b" />
                  <Tooltip 
                    contentStyle={{ 
                      backgroundColor: 'white', 
                      border: '1px solid #e2e8f0',
                      borderRadius: '8px'
                    }} 
                  />
                  <Bar dataKey="hours" fill="#3b82f6" radius={[8, 8, 0, 0]} />
                </BarChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>
        </TabsContent>

        <TabsContent value="subjects">
          <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60">
            <CardHeader>
              <CardTitle className="flex items-center gap-2 text-slate-900 dark:text-slate-100">
                <BookOpen className="w-5 h-5 text-blue-500" />
                Study Time by Subject
              </CardTitle>
            </CardHeader>
            <CardContent>
              <ResponsiveContainer width="100%" height={300}>
                <PieChart>
                  <Pie
                    data={subjectData}
                    cx="50%"
                    cy="50%"
                    labelLine={false}
                    label={({ name, value }) => `${name}: ${value}h`}
                    outerRadius={80}
                    fill="#8884d8"
                    dataKey="value"
                  >
                    {subjectData.map((entry, index) => (
                      <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                    ))}
                  </Pie>
                  <Tooltip />
                </PieChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>
        </TabsContent>

        <TabsContent value="productivity">
          <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60">
            <CardHeader>
              <CardTitle className="flex items-center gap-2 text-slate-900 dark:text-slate-100">
                <TrendingUp className="w-5 h-5 text-green-500" />
                Productivity Trend
              </CardTitle>
            </CardHeader>
            <CardContent>
              <ResponsiveContainer width="100%" height={300}>
                <LineChart data={productivityData}>
                  <CartesianGrid strokeDasharray="3 3" stroke="#e2e8f0" />
                  <XAxis dataKey="date" stroke="#64748b" />
                  <YAxis domain={[0, 5]} stroke="#64748b" />
                  <Tooltip 
                    contentStyle={{ 
                      backgroundColor: 'white', 
                      border: '1px solid #e2e8f0',
                      borderRadius: '8px'
                    }} 
                  />
                  <Line 
                    type="monotone" 
                    dataKey="productivity" 
                    stroke="#10b981" 
                    strokeWidth={3}
                    dot={{ fill: '#10b981', r: 5 }}
                  />
                </LineChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>
        </TabsContent>
      </Tabs>
    </div>
  );
}import React, { useState } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Badge } from "@/components/ui/badge";
import { Progress } from "@/components/ui/progress";
import { 
  Target, 
  Plus, 
  Calendar, 
  Clock, 
  MapPin, 
  Trash2,
  AlertCircle
} from "lucide-react";
import { format, differenceInDays } from "date-fns";

export default function Exams() {
  const [showCreateForm, setShowCreateForm] = useState(false);
  const [formData, setFormData] = useState({
    subject: "",
    exam_name: "",
    exam_date: "",
    exam_time: "",
    location: "",
    preparation_status: "not_started",
    topics_to_cover: [],
    target_grade: ""
  });
  const [topicInput, setTopicInput] = useState("");

  const queryClient = useQueryClient();

  const { data: exams = [], isLoading } = useQuery({
    queryKey: ['exams'],
    queryFn: () => base44.entities.Exam.list("exam_date", 50),
  });

  const createMutation = useMutation({
    mutationFn: (data) => base44.entities.Exam.create(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['exams'] });
      setShowCreateForm(false);
      setFormData({ subject: "", exam_name: "", exam_date: "", exam_time: "", location: "", preparation_status: "not_started", topics_to_cover: [], target_grade: "" });
    },
  });

  const deleteMutation = useMutation({
    mutationFn: (id) => base44.entities.Exam.delete(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['exams'] });
    },
  });

  const updateMutation = useMutation({
    mutationFn: ({ id, data }) => base44.entities.Exam.update(id, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['exams'] });
    },
  });

  const handleCreateExam = () => {
    createMutation.mutate(formData);
  };

  const addTopic = () => {
    if (topicInput.trim()) {
      setFormData({
        ...formData,
        topics_to_cover: [...formData.topics_to_cover, topicInput.trim()]
      });
      setTopicInput("");
    }
  };

  const removeTopic = (index) => {
    setFormData({
      ...formData,
      topics_to_cover: formData.topics_to_cover.filter((_, i) => i !== index)
    });
  };

  const getStatusColor = (status) => {
    switch (status) {
      case 'ready': return 'bg-green-100 text-green-800 dark:bg-green-950/50 dark:text-green-400';
      case 'well_prepared': return 'bg-blue-100 text-blue-800 dark:bg-blue-950/50 dark:text-blue-400';
      case 'in_progress': return 'bg-yellow-100 text-yellow-800 dark:bg-yellow-950/50 dark:text-yellow-400';
      default: return 'bg-red-100 text-red-800 dark:bg-red-950/50 dark:text-red-400';
    }
  };

  const getStatusProgress = (status) => {
    switch (status) {
      case 'ready': return 100;
      case 'well_prepared': return 75;
      case 'in_progress': return 50;
      default: return 25;
    }
  };

  const upcomingExams = exams.filter(exam => new Date(exam.exam_date) >= new Date());
  const pastExams = exams.filter(exam => new Date(exam.exam_date) < new Date());

  return (
    <div className="p-6 max-w-7xl mx-auto">
      <div className="flex flex-col md:flex-row justify-between items-start md:items-center mb-8 gap-4">
        <div>
          <h1 className="text-3xl font-bold text-slate-900 dark:text-slate-50">Exams</h1>
          <p className="text-slate-600 dark:text-slate-400 mt-1">Track your upcoming exams and preparation</p>
        </div>
        <Button
          onClick={() => setShowCreateForm(true)}
          className="bg-gradient-to-r from-blue-500 to-indigo-600 hover:from-blue-600 hover:to-indigo-700 shadow-lg"
        >
          <Plus className="w-4 h-4 mr-2" />
          Add Exam
        </Button>
      </div>

      {/* Upcoming Exams */}
      <div className="mb-8">
        <h2 className="text-xl font-bold text-slate-900 dark:text-slate-100 mb-4">Upcoming Exams</h2>
        <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
          {upcomingExams.map(exam => {
            const daysLeft = differenceInDays(new Date(exam.exam_date), new Date());
            const isUrgent = daysLeft <= 7;

            return (
              <Card key={exam.id} className={`border-none shadow-lg bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60 ${isUrgent ? 'ring-2 ring-red-500 dark:ring-red-600' : ''}`}>
                <CardHeader>
                  <div className="flex justify-between items-start">
                    <div className="flex-1">
                      <CardTitle className="text-slate-900 dark:text-slate-100">{exam.exam_name}</CardTitle>
                      <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">{exam.subject}</p>
                    </div>
                    <Button
                      variant="ghost"
                      size="icon"
                      onClick={() => deleteMutation.mutate(exam.id)}
                      className="text-red-500 hover:text-red-700 dark:text-red-400"
                    >
                      <Trash2 className="w-4 h-4" />
                    </Button>
                  </div>
                </CardHeader>
                <CardContent>
                  <div className="space-y-4">
                    {isUrgent && (
                      <div className="flex items-center gap-2 p-2 bg-red-50 dark:bg-red-950/30 rounded-lg">
                        <AlertCircle className="w-4 h-4 text-red-600 dark:text-red-400" />
                        <span className="text-sm font-medium text-red-600 dark:text-red-400">Exam in {daysLeft} day{daysLeft !== 1 ? 's' : ''}!</span>
                      </div>
                    )}

                    <div className="flex items-center gap-2 text-sm text-slate-600 dark:text-slate-400">
                      <Calendar className="w-4 h-4 text-blue-500" />
                      <span>{format(new Date(exam.exam_date), "MMMM d, yyyy")}</span>
                    </div>

                    {exam.exam_time && (
                      <div className="flex items-center gap-2 text-sm text-slate-600 dark:text-slate-400">
                        <Clock className="w-4 h-4 text-green-500" />
                        <span>{exam.exam_time}</span>
                      </div>
                    )}

                    {exam.location && (
                      <div className="flex items-center gap-2 text-sm text-slate-600 dark:text-slate-400">
                        <MapPin className="w-4 h-4 text-orange-500" />
                        <span>{exam.location}</span>
                      </div>
                    )}

                    <div>
                      <div className="flex justify-between items-center mb-2">
                        <span className="text-sm text-slate-600 dark:text-slate-400">Preparation</span>
                        <Badge className={getStatusColor(exam.preparation_status)}>
                          {exam.preparation_status.replace('_', ' ')}
                        </Badge>
                      </div>
                      <Progress value={getStatusProgress(exam.preparation_status)} className="h-2" />
                    </div>

                    {exam.target_grade && (
                      <div className="flex justify-between items-center p-2 bg-slate-50 dark:bg-slate-700/50 rounded-lg">
                        <span className="text-sm text-slate-600 dark:text-slate-400">Target Grade</span>
                        <span className="font-bold text-slate-900 dark:text-slate-100">{exam.target_grade}</span>
                      </div>
                    )}

                    {exam.topics_to_cover && exam.topics_to_cover.length > 0 && (
                      <div>
                        <p className="text-sm font-medium text-slate-700 dark:text-slate-300 mb-2">Topics to Cover</p>
                        <div className="flex flex-wrap gap-2">
                          {exam.topics_to_cover.slice(0, 3).map((topic, index) => (
                            <Badge key={index} variant="outline" className="text-xs dark:border-slate-600 dark:text-slate-300">
                              {topic}
                            </Badge>
                          ))}
                          {exam.topics_to_cover.length > 3 && (
                            <Badge variant="outline" className="text-xs dark:border-slate-600 dark:text-slate-300">
                              +{exam.topics_to_cover.length - 3} more
                            </Badge>
                          )}
                        </div>
                      </div>
                    )}

                    <Select 
                      value={exam.preparation_status} 
                      onValueChange={(value) => updateMutation.mutate({ id: exam.id, data: { preparation_status: value } })}
                    >
                      <SelectTrigger className="bg-white dark:bg-slate-900 border-slate-300 dark:border-slate-700">
                        <SelectValue />
                      </SelectTrigger>
                      <SelectContent className="bg-white dark:bg-slate-800">
                        <SelectItem value="not_started">Not Started</SelectItem>
                        <SelectItem value="in_progress">In Progress</SelectItem>
                        <SelectItem value="well_prepared">Well Prepared</SelectItem>
                        <SelectItem value="ready">Ready</SelectItem>
                      </SelectContent>
                    </Select>
                  </div>
                </CardContent>
              </Card>
            );
          })}
        </div>

        {upcomingExams.length === 0 && (
          <Card className="border-none shadow-lg bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60">
            <CardContent className="py-16 text-center">
              <Target className="w-16 h-16 text-slate-300 dark:text-slate-700 mx-auto mb-4" />
              <h3 className="text-xl font-semibold text-slate-900 dark:text-slate-100 mb-2">No upcoming exams</h3>
              <p className="text-slate-600 dark:text-slate-400">Add your first exam to start tracking!</p>
            </CardContent>
          </Card>
        )}
      </div>

      {/* Past Exams */}
      {pastExams.length > 0 && (
        <div>
          <h2 className="text-xl font-bold text-slate-900 dark:text-slate-100 mb-4">Past Exams</h2>
          <div className="space-y-3">
            {pastExams.map(exam => (
              <Card key={exam.id} className="border-none shadow-lg bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60 opacity-60">
                <CardContent className="p-4">
                  <div className="flex justify-between items-center">
                    <div>
                      <h3 className="font-semibold text-slate-900 dark:text-slate-100">{exam.exam_name}</h3>
                      <p className="text-sm text-slate-600 dark:text-slate-400">{exam.subject} - {format(new Date(exam.exam_date), "MMM d, yyyy")}</p>
                    </div>
                    <Button
                      variant="ghost"
                      size="icon"
                      onClick={() => deleteMutation.mutate(exam.id)}
                      className="text-red-500 hover:text-red-700 dark:text-red-400"
                    >
                      <Trash2 className="w-4 h-4" />
                    </Button>
                  </div>
                </CardContent>
              </Card>
            ))}
          </div>
        </div>
      )}

      {/* Create Form Modal */}
      {showCreateForm && (
        <div className="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center p-4 z-50 overflow-y-auto">
          <Card className="max-w-md w-full my-8 bg-white dark:bg-slate-800">
            <CardHeader>
              <CardTitle className="text-slate-900 dark:text-slate-100">Add New Exam</CardTitle>
            </CardHeader>
            <CardContent className="space-y-4">
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Exam Name</label>
                <Input
                  value={formData.exam_name}
                  onChange={(e) => setFormData({...formData, exam_name: e.target.value})}
                  placeholder="e.g., Midterm Examination"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Subject</label>
                <Input
                  value={formData.subject}
                  onChange={(e) => setFormData({...formData, subject: e.target.value})}
                  placeholder="e.g., Physics"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div className="grid grid-cols-2 gap-4">
                <div>
                  <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Exam Date</label>
                  <Input
                    type="date"
                    value={formData.exam_date}
                    onChange={(e) => setFormData({...formData, exam_date: e.target.value})}
                    className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                  />
                </div>
                <div>
                  <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Time</label>
                  <Input
                    value={formData.exam_time}
                    onChange={(e) => setFormData({...formData, exam_time: e.target.value})}
                    placeholder="9:00 AM"
                    className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                  />
                </div>
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Location</label>
                <Input
                  value={formData.location}
                  onChange={(e) => setFormData({...formData, location: e.target.value})}
                  placeholder="Room 101"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Target Grade</label>
                <Input
                  value={formData.target_grade}
                  onChange={(e) => setFormData({...formData, target_grade: e.target.value})}
                  placeholder="A"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                />
              </div>
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Topics to Cover</label>
                <div className="flex gap-2 mt-1">
                  <Input
                    value={topicInput}
                    onChange={(e) => setTopicInput(e.target.value)}
                    onKeyPress={(e) => e.key === 'Enter' && (e.preventDefault(), addTopic())}
                    placeholder="Add a topic"
                    className="bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                  />
                  <Button type="button" onClick={addTopic} className="bg-blue-500 hover:bg-blue-600">
                    Add
                  </Button>
                </div>
                <div className="flex flex-wrap gap-2 mt-2">
                  {formData.topics_to_cover.map((topic, index) => (
                    <Badge key={index} className="bg-blue-100 text-blue-800 dark:bg-blue-950/50 dark:text-blue-400">
                      {topic}
                      <button onClick={() => removeTopic(index)} className="ml-2">×</button>
                    </Badge>
                  ))}
                </div>
              </div>
              <div className="flex gap-3 pt-4">
                <Button variant="outline" onClick={() => setShowCreateForm(false)} className="flex-1 dark:bg-slate-700 dark:text-slate-300 dark:border-slate-600">
                  Cancel
                </Button>
                <Button onClick={handleCreateExam} className="flex-1 bg-gradient-to-r from-blue-500 to-indigo-600">
                  Add Exam
                </Button>
              </div>
            </CardContent>
          </Card>
        </div>
      )}
    </div>
  );
}import React, { useState } from "react";
import { base44 } from "@/api/base44Client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { Slider } from "@/components/ui/slider";
import { Heart, Zap, TrendingDown, Target, Plus, Calendar } from "lucide-react";
import { format } from "date-fns";
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from "recharts";

export default function MoodTracker() {
  const [showForm, setShowForm] = useState(false);
  const [formData, setFormData] = useState({
    mood_score: 5,
    energy_level: 5,
    stress_level: 5,
    motivation: 5,
    notes: "",
    date: format(new Date(), 'yyyy-MM-dd')
  });

  const queryClient = useQueryClient();

  const { data: moodEntries = [], isLoading } = useQuery({
    queryKey: ['moods'],
    queryFn: () => base44.entities.MoodEntry.list("-date", 30),
  });

  const createMutation = useMutation({
    mutationFn: (data) => base44.entities.MoodEntry.create(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['moods'] });
      setShowForm(false);
      setFormData({ mood_score: 5, energy_level: 5, stress_level: 5, motivation: 5, notes: "", date: format(new Date(), 'yyyy-MM-dd') });
    },
  });

  const handleSubmit = () => {
    createMutation.mutate(formData);
  };

  const getChartData = () => {
    return moodEntries
      .slice(0, 14)
      .reverse()
      .map(entry => ({
        date: format(new Date(entry.date), 'MMM dd'),
        mood: entry.mood_score,
        energy: entry.energy_level,
        stress: entry.stress_level,
        motivation: entry.motivation
      }));
  };

  const getAverages = () => {
    if (moodEntries.length === 0) return { mood: 0, energy: 0, stress: 0, motivation: 0 };
    
    const sum = moodEntries.reduce((acc, entry) => ({
      mood: acc.mood + entry.mood_score,
      energy: acc.energy + entry.energy_level,
      stress: acc.stress + entry.stress_level,
      motivation: acc.motivation + entry.motivation
    }), { mood: 0, energy: 0, stress: 0, motivation: 0 });

    return {
      mood: (sum.mood / moodEntries.length).toFixed(1),
      energy: (sum.energy / moodEntries.length).toFixed(1),
      stress: (sum.stress / moodEntries.length).toFixed(1),
      motivation: (sum.motivation / moodEntries.length).toFixed(1)
    };
  };

  const getMoodEmoji = (score) => {
    if (score >= 8) return "😄";
    if (score >= 6) return "🙂";
    if (score >= 4) return "😐";
    if (score >= 2) return "😕";
    return "😢";
  };

  const chartData = getChartData();
  const averages = getAverages();

  return (
    <div className="p-6 max-w-7xl mx-auto">
      <div className="flex flex-col md:flex-row justify-between items-start md:items-center mb-8 gap-4">
        <div>
          <h1 className="text-3xl font-bold text-slate-900 dark:text-slate-50">Mood Tracker</h1>
          <p className="text-slate-600 dark:text-slate-400 mt-1">Track your emotional wellbeing and study balance</p>
        </div>
        <Button
          onClick={() => setShowForm(true)}
          className="bg-gradient-to-r from-purple-500 to-pink-600 hover:from-purple-600 hover:to-pink-700 shadow-lg"
        >
          <Plus className="w-4 h-4 mr-2" />
          Log Today's Mood
        </Button>
      </div>

      {/* Averages */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
        <Card className="border-none shadow-lg bg-gradient-to-br from-purple-50 to-pink-50 dark:from-slate-800 dark:to-purple-950/30 dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">Avg Mood</CardTitle>
              <Heart className="w-5 h-5 text-purple-500" />
            </div>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">{averages.mood}/10</div>
            <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">Overall wellbeing</p>
          </CardContent>
        </Card>

        <Card className="border-none shadow-lg bg-gradient-to-br from-yellow-50 to-orange-50 dark:from-slate-800 dark:to-orange-950/30 dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">Avg Energy</CardTitle>
              <Zap className="w-5 h-5 text-yellow-500" />
            </div>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">{averages.energy}/10</div>
            <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">Energy levels</p>
          </CardContent>
        </Card>

        <Card className="border-none shadow-lg bg-gradient-to-br from-red-50 to-pink-50 dark:from-slate-800 dark:to-red-950/30 dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">Avg Stress</CardTitle>
              <TrendingDown className="w-5 h-5 text-red-500" />
            </div>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">{averages.stress}/10</div>
            <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">Stress levels</p>
          </CardContent>
        </Card>

        <Card className="border-none shadow-lg bg-gradient-to-br from-blue-50 to-indigo-50 dark:from-slate-800 dark:to-blue-950/30 dark:border dark:border-slate-700/60">
          <CardHeader className="pb-2">
            <div className="flex items-center justify-between">
              <CardTitle className="text-sm font-medium text-slate-700 dark:text-slate-300">Avg Motivation</CardTitle>
              <Target className="w-5 h-5 text-blue-500" />
            </div>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold text-slate-900 dark:text-slate-50">{averages.motivation}/10</div>
            <p className="text-sm text-slate-600 dark:text-slate-400 mt-1">Drive to study</p>
          </CardContent>
        </Card>
      </div>

      {/* Chart */}
      <Card className="border-none shadow-lg mb-8 bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60">
        <CardHeader>
          <CardTitle className="text-slate-900 dark:text-slate-100">14-Day Mood Trend</CardTitle>
        </CardHeader>
        <CardContent>
          <ResponsiveContainer width="100%" height={300}>
            <LineChart data={chartData}>
              <CartesianGrid strokeDasharray="3 3" stroke="#e2e8f0" />
              <XAxis dataKey="date" stroke="#64748b" />
              <YAxis domain={[0, 10]} stroke="#64748b" />
              <Tooltip 
                contentStyle={{ 
                  backgroundColor: 'white',
                  border: '1px solid #e2e8f0',
                  borderRadius: '8px'
                }} 
              />
              <Line type="monotone" dataKey="mood" stroke="#a855f7" strokeWidth={2} name="Mood" />
              <Line type="monotone" dataKey="energy" stroke="#f59e0b" strokeWidth={2} name="Energy" />
              <Line type="monotone" dataKey="stress" stroke="#ef4444" strokeWidth={2} name="Stress" />
              <Line type="monotone" dataKey="motivation" stroke="#3b82f6" strokeWidth={2} name="Motivation" />
            </LineChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>

      {/* Recent Entries */}
      <div>
        <h2 className="text-xl font-bold text-slate-900 dark:text-slate-100 mb-4">Recent Entries</h2>
        <div className="space-y-4">
          {moodEntries.slice(0, 5).map(entry => (
            <Card key={entry.id} className="border-none shadow-lg bg-white dark:bg-slate-800 dark:border dark:border-slate-700/60">
              <CardContent className="p-6">
                <div className="flex items-start justify-between gap-4">
                  <div className="flex items-center gap-4">
                    <div className="text-4xl">{getMoodEmoji(entry.mood_score)}</div>
                    <div>
                      <div className="flex items-center gap-2 mb-2">
                        <Calendar className="w-4 h-4 text-slate-500 dark:text-slate-400" />
                        <span className="font-medium text-slate-900 dark:text-slate-100">
                          {format(new Date(entry.date), "MMMM d, yyyy")}
                        </span>
                      </div>
                      <div className="grid grid-cols-2 md:grid-cols-4 gap-4 text-sm">
                        <div>
                          <span className="text-slate-600 dark:text-slate-400">Mood:</span>
                          <span className="ml-2 font-semibold text-purple-600 dark:text-purple-400">{entry.mood_score}/10</span>
                        </div>
                        <div>
                          <span className="text-slate-600 dark:text-slate-400">Energy:</span>
                          <span className="ml-2 font-semibold text-yellow-600 dark:text-yellow-400">{entry.energy_level}/10</span>
                        </div>
                        <div>
                          <span className="text-slate-600 dark:text-slate-400">Stress:</span>
                          <span className="ml-2 font-semibold text-red-600 dark:text-red-400">{entry.stress_level}/10</span>
                        </div>
                        <div>
                          <span className="text-slate-600 dark:text-slate-400">Motivation:</span>
                          <span className="ml-2 font-semibold text-blue-600 dark:text-blue-400">{entry.motivation}/10</span>
                        </div>
                      </div>
                      {entry.notes && (
                        <p className="text-sm text-slate-600 dark:text-slate-400 mt-3 italic">"{entry.notes}"</p>
                      )}
                    </div>
                  </div>
                </div>
              </CardContent>
            </Card>
          ))}
        </div>
      </div>

      {/* Form Modal */}
      {showForm && (
        <div className="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center p-4 z-50">
          <Card className="max-w-md w-full bg-white dark:bg-slate-800">
            <CardHeader>
              <CardTitle className="text-slate-900 dark:text-slate-100">Log Your Mood</CardTitle>
            </CardHeader>
            <CardContent className="space-y-6">
              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300 flex items-center justify-between mb-2">
                  <span>Mood Score</span>
                  <span className="text-2xl">{getMoodEmoji(formData.mood_score)}</span>
                </label>
                <Slider
                  value={[formData.mood_score]}
                  onValueChange={(value) => setFormData({...formData, mood_score: value[0]})}
                  max={10}
                  min={1}
                  step={1}
                  className="mt-2"
                />
                <div className="flex justify-between text-xs text-slate-500 dark:text-slate-400 mt-1">
                  <span>1 - Very Bad</span>
                  <span className="font-bold text-purple-600 dark:text-purple-400">{formData.mood_score}</span>
                  <span>10 - Excellent</span>
                </div>
              </div>

              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300 flex items-center justify-between mb-2">
                  <span>Energy Level</span>
                  <Zap className="w-4 h-4 text-yellow-500" />
                </label>
                <Slider
                  value={[formData.energy_level]}
                  onValueChange={(value) => setFormData({...formData, energy_level: value[0]})}
                  max={10}
                  min={1}
                  step={1}
                />
                <div className="text-center text-sm font-bold text-yellow-600 dark:text-yellow-400 mt-1">{formData.energy_level}/10</div>
              </div>

              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300 flex items-center justify-between mb-2">
                  <span>Stress Level</span>
                  <TrendingDown className="w-4 h-4 text-red-500" />
                </label>
                <Slider
                  value={[formData.stress_level]}
                  onValueChange={(value) => setFormData({...formData, stress_level: value[0]})}
                  max={10}
                  min={1}
                  step={1}
                />
                <div className="text-center text-sm font-bold text-red-600 dark:text-red-400 mt-1">{formData.stress_level}/10</div>
              </div>

              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300 flex items-center justify-between mb-2">
                  <span>Motivation</span>
                  <Target className="w-4 h-4 text-blue-500" />
                </label>
                <Slider
                  value={[formData.motivation]}
                  onValueChange={(value) => setFormData({...formData, motivation: value[0]})}
                  max={10}
                  min={1}
                  step={1}
                />
                <div className="text-center text-sm font-bold text-blue-600 dark:text-blue-400 mt-1">{formData.motivation}/10</div>
              </div>

              <div>
                <label className="text-sm font-medium text-slate-700 dark:text-slate-300">Notes (Optional)</label>
                <Textarea
                  value={formData.notes}
                  onChange={(e) => setFormData({...formData, notes: e.target.value})}
                  placeholder="How are you feeling today? Any thoughts?"
                  className="mt-1 bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-100 border-slate-300 dark:border-slate-700"
                  rows={3}
                />
              </div>

              <div className="flex gap-3 pt-4">
                <Button variant="outline" onClick={() => setShowForm(false)} className="flex-1 dark:bg-slate-700 dark:text-slate-300 dark:border-slate-600">
                  Cancel
                </Button>
                <Button onClick={handleSubmit} className="flex-1 bg-gradient-to-r from-purple-500 to-pink-600">
                  Save Entry
                </Button>
              </div>
            </CardContent>
          </Card>
        </div>
      )}
    </div>
  );
}

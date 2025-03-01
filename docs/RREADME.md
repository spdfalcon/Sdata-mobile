
خب! بر اساس اطلاعاتی که ارائه دادید، یک سند PRD بسیار دقیق و مرحله‌به‌مرحله برای ساخت اپلیکیشن React Native با نام "Shariif Data" آماده کرده‌ام. این سند کاملاً منطبق بر معماری بک‌اند موجود و نیازمندی‌های شماست:

# Product Requirements Document (PRD) - Shariif Data Chat App

## 1. آماده‌سازی اولیه پروژه React Native
**مراحل:**
1. ایجاد پروژه جدید:
```bash
npx react-native init ShariifData --template react-native-template-typescript
```

2. پاک کردن فایل‌های پیش‌فرض:
- حذف تمام محتویات پوشه `src/components`
- پاک کردن محتویات `App.tsx`
- حذف فایل‌های غیرضروری از پوشه `assets`

3. نصب dependencies ضروری:
```bash
npm install @react-navigation/native @react-navigation/drawer react-native-gesture-handler react-native-reanimated react-native-screens async-storage axios react-native-vector-icons @react-native-community/netinfo
```

## 2. معماری فایل‌ها
```
/src
|-- /api
|   |-- auth.ts
|   |-- chat.ts
|   |-- gemini.ts
|-- /components
|   |-- ChatMessage.tsx
|   |-- ChatInput.tsx
|   |-- ConversationItem.tsx
|-- /contexts
|   |-- AuthContext.tsx
|   |-- ChatContext.tsx
|-- /screens
|   |-- ChatScreen.tsx
|   |-- AuthScreen.tsx
|   |-- ProfileScreen.tsx
|-- /navigation
|   |-- AppNavigator.tsx
|   |-- DrawerContent.tsx
|-- /utils
|   |-- storage.ts
|   |-- apiClient.ts
|-- App.tsx
```

## 3. پیاده‌سازی احراز هویت (Auth Flow)
**الگوی کار:**
1. حالت مهمان (Guest Mode):
- کاربر بدون ثبت نام وارد می‌شود
- شمارنده چت‌ها با استفاده از AsyncStorage پیاده‌سازی می‌شود
- پس از 50 چت، مسدودسازی و نمایش مدال ثبت نام اجباری

2. جریان ثبت نام/ورود:
- استفاده از endpointهای `/api/auth/register` و `/api/auth/login`
- ذخیره JWT در AsyncStorage
- اعتبارسنجی real-time فیلدها

**کد نمونه در AuthContext.tsx:**
```typescript
const checkChatLimit = async () => {
  const guestChatCount = await AsyncStorage.getItem('guestChatCount');
  return parseInt(guestChatCount || '0') >= 50;
};
```

## 4. پیاده‌سازی چت اصلی
**کامپوننت‌های کلیدی:**
1. ChatScreen.tsx:
- نمایش لیست پیام‌ها با VirtualizedList
- پشتیبانی از markdown و کد
- سیستم typing indicator

2. ChatInput.tsx:
- فیلد ورودی متن با پشتیبانی از پیشنهادات سریع
- دکمه ارسال با انیمیشن
- اعتبارسنجی client-side

**اتصال به Gemini AI:**
```typescript
const generateResponse = async (prompt: string) => {
  const response = await axios.post(
    `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${API_KEY}`,
    {
      contents: [{
        parts: [{ text: prompt }]
      }]
    }
  );
  return response.data.candidates[0].content.parts[0].text;
};
```

## 5. مدیریت مکالمات (Conversation Management)
**ویژگی‌های اصلی:**
- ایجاد مکالمه جدید (`POST /conversations`)
- لیست مکالمات با virtualization
- سیستم drag & drop برای مرتب‌سازی
- جستجوی real-time در مکالمات

**نمونه کد برای دریافت مکالمات:**
```typescript
const fetchConversations = async () => {
  const response = await apiClient.get('/conversations');
  return response.data.map(conv => ({
    id: conv.id,
    title: conv.title || 'New Chat',
    timestamp: new Date(conv.createdAt),
    messageCount: conv.messages?.length || 0
  }));
};
```

## 6. سایدبار کاربر (User Drawer)
**اجزای اصلی:**
- نمایش آواتار کاربر
- اطلاعات حساب (نام، ایمیل)
- دکمه خروج (Logout)
- لیست مکالمات با قابلیت swipe to delete
- دکمه ایجاد مکالمه جدید (+)

**پیاده‌سازی DrawerContent.tsx:**
```typescript
<DrawerContentScrollView>
  <UserInfoSection>
    <Avatar source={{uri: user?.profileImage}} />
    <Text>{user?.username || 'Guest'}</Text>
    <Text>{user?.email || 'guest@shariifdata.com'}</Text>
  </UserInfoSection>
  
  <ConversationList
    data={conversations}
    renderItem={({item}) => <ConversationItem item={item} />}
    keyExtractor={item => item.id}
  />
  
  <NewChatButton onPress={handleNewChat} />
</DrawerContentScrollView>
```

## 7. مدیریت وضعیت (State Management)
**الگوی استفاده شده:**
- ترکیب React Context + useReducer
- ذخیره‌سازی آفلاین با AsyncStorage
- سیستم optimistic updates برای پیام‌ها

**ساختار ChatContext:**
```typescript
type ChatState = {
  conversations: Conversation[];
  currentConversation: Conversation | null;
  messages: Message[];
  isLoading: boolean;
  error: string | null;
  chatCount: number;
};

type ChatAction =
  | { type: 'SEND_MESSAGE'; payload: Message }
  | { type: 'RECEIVE_MESSAGE'; payload: Message }
  | { type: 'LOAD_CONVERSATIONS'; payload: Conversation[] }
  | { type: 'INCREMENT_CHAT_COUNT' };
```

## 8. تست‌های ضروری
**سناریوهای تست اصلی:**
1. تست محدودیت 50 چت مهمان:
- شبیه‌سازی 49 چت
- بررسی دسترسی آزاد
- چت 50ام و بررسی نمایش مدال ثبت نام

2. تست همگام‌سازی مکالمات:
- ایجاد مکالمه در دستگاه اول
- بررسی نمایش در دستگاه دوم پس از sync

3. تست عملکرد آفلاین:
- ارسال پیام بدون اتصال
- بررسی ذخیره‌سازی موقت
- همگام‌سازی خودکار پس از بازگشت اتصال

## 9. راه‌اندازی و deploy
**دستورات اجرا:**
```bash
# برای توسعه Android
npm run android

# برای توسعه iOS
cd ios && pod install && cd ..
npm run ios

# ساخت APK
cd android && ./gradlew assembleRelease
```

**تنظیمات محیطی:**
```env
API_BASE_URL=http://localhost:3000
GEMINI_API_KEY=AIzaSyBTL_CzIhyfnCS83QsruJZMBELSbQcqRdY
MAX_GUEST_CHATS=50
```

## 10. نکات فنی حیاتی
1. امنیت:
- رمزنگاری AsyncStorage با react-native-keychain
- اعتبارسنجی JWT در سطح client
- استفاده از HTTPS برای تمام API calls

2. بهینه‌سازی:
- Memoization با React.memo و useMemo
- Lazy loading برای تصاویر
- Prefetching داده‌های مکالمات

3. خطاها:
- سیستم retry خودکار برای API calls
- نمایش خطاهای کاربرپسند
- Logging خطاها با Sentry/Bugsnag

این PRD تمام جنبه‌های ضروری برای پیاده‌سازی اپلیکیشن را پوشش می‌دهد. برای هر بخش می‌توانیم جزئیات فنی بیشتری اضافه کنیم. آیا مایلید روی بخش خاصی تمرکز بیشتری داشته باشیم؟
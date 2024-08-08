import 'package:flutter/material.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';

void main() {
  runApp(ReminderApp());
}

class ReminderApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Reminder App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: ReminderHomePage(),
    );
  }
}

class ReminderHomePage extends StatefulWidget {
  @override
  _ReminderHomePageState createState() => _ReminderHomePageState();
}

class _ReminderHomePageState extends State<ReminderHomePage> {
  final List<String> daysOfWeek = [
    'Monday',
    'Tuesday',
    'Wednesday',
    'Thursday',
    'Friday',
    'Saturday',
    'Sunday'
  ];

  final List<String> activities = [
    'Wake up',
    'Go to gym',
    'Breakfast',
    'Meetings',
    'Lunch',
    'Quick nap',
    'Go to library',
    'Dinner',
    'Go to sleep',
  ];

  String selectedDay = 'Monday';
  String selectedTime = '08:00 AM';
  String selectedActivity = 'Wake up';

  TimeOfDay selectedTimeOfDay = TimeOfDay(hour: 8, minute: 0);

  FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin;

  @override
  void initState() {
    super.initState();
    flutterLocalNotificationsPlugin = FlutterLocalNotificationsPlugin();
    var initializationSettingsAndroid = AndroidInitializationSettings('@mipmap/ic_launcher');
    var initializationSettingsIOS = IOSInitializationSettings();
    var initializationSettings = InitializationSettings(android: initializationSettingsAndroid, iOS: initializationSettingsIOS);
    flutterLocalNotificationsPlugin.initialize(initializationSettings);
  }

  Future<void> _scheduleNotification() async {
    var scheduledTime = Time(selectedTimeOfDay.hour, selectedTimeOfDay.minute, 0);

    var androidDetails = AndroidNotificationDetails(
      'reminder_channel_id',
      'Reminders',
      channelDescription: 'Channel for reminder notifications',
      importance: Importance.high,
      priority: Priority.high,
    );
    var iOSDetails = IOSNotificationDetails();
    var notificationDetails = NotificationDetails(android: androidDetails, iOS: iOSDetails);

    await flutterLocalNotificationsPlugin.showDailyAtTime(
      0,
      'Reminder: $selectedActivity',
      'It\'s time to $selectedActivity!',
      scheduledTime,
      notificationDetails,
    );
  }

  _pickTime() async {
    TimeOfDay time = await showTimePicker(
      context: context,
      initialTime: selectedTimeOfDay,
    );
    if (time != null) {
      setState(() {
        selectedTimeOfDay = time;
        selectedTime = time.format(context);
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Reminder App'),
      ),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          children: [
            DropdownButton<String>(
              value: selectedDay,
              onChanged: (String newValue) {
                setState(() {
                  selectedDay = newValue;
                });
              },
              items: daysOfWeek.map<DropdownMenuItem<String>>((String value) {
                return DropdownMenuItem<String>(
                  value: value,
                  child: Text(value),
                );
              }).toList(),
            ),
            SizedBox(height: 16.0),
            GestureDetector(
              onTap: _pickTime,
              child: InputDecorator(
                decoration: InputDecoration(
                  labelText: 'Select Time',
                  border: OutlineInputBorder(),
                ),
                child: Text(selectedTime),
              ),
            ),
            SizedBox(height: 16.0),
            DropdownButton<String>(
              value: selectedActivity,
              onChanged: (String newValue) {
                setState(() {
                  selectedActivity = newValue;
                });
              },
              items: activities.map<DropdownMenuItem<String>>((String value) {
                return DropdownMenuItem<String>(
                  value: value,
                  child: Text(value),
                );
              }).toList(),
            ),
            SizedBox(height: 16.0),
            ElevatedButton(
              onPressed: _scheduleNotification,
              child: Text('Set Reminder'),
            ),
          ],
        ),
      ),
    );
  }
}

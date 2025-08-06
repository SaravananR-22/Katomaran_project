# Katomaran_project
import 'package:flutter/material.dart';

void main() {
  runApp(const MainApp());
}

class MainApp extends StatelessWidget {
  const MainApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'To-Do List App',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.indigo,
        scaffoldBackgroundColor: Colors.grey[100],
        appBarTheme: const AppBarTheme(
          backgroundColor: Colors.indigo,
          foregroundColor: Colors.white,
          centerTitle: true,
          elevation: 2,
        ),
        inputDecorationTheme: const InputDecorationTheme(
          border: OutlineInputBorder(),
        ),
      ),
      home: const ToDoHomeScreen(),
    );
  }
}

class ToDoHomeScreen extends StatefulWidget {
  const ToDoHomeScreen({super.key});

  @override
  State<ToDoHomeScreen> createState() => _ToDoHomeScreenState();
}

class _ToDoHomeScreenState extends State<ToDoHomeScreen> {
  final TextEditingController titleController = TextEditingController();
  final TextEditingController descriptionController = TextEditingController();

  String? selectedPriority;
  TimeOfDay? selectedTime;
  DateTime? selectedDate;
  List<Map<String, String>> items = [];
  int? editingIndex;

  void resetForm() {
    titleController.clear();
    descriptionController.clear();
    selectedPriority = null;
    selectedTime = null;
    selectedDate = null;
    editingIndex = null;
  }

  void showErrorSnack(String message) {
    ScaffoldMessenger.of(context).showSnackBar(SnackBar(
      content: Text(message),
      backgroundColor: Colors.red,
    ));
  }

  void handleSave() {
    if (titleController.text.isEmpty ||
        selectedPriority == null ||
        selectedTime == null ||
        selectedDate == null) {
      showErrorSnack("Please fill in title, select priority, date and time.");
      return;
    }

    final newItem = {
      'title': titleController.text,
      'description': descriptionController.text,
      'priority': selectedPriority!,
      'time': selectedTime!.format(context),
      'date': "${selectedDate!.day}/${selectedDate!.month}/${selectedDate!.year}",
    };

    setState(() {
      if (editingIndex == null) {
        items.add(newItem);
      } else {
        items[editingIndex!] = newItem;
        editingIndex = null;
      }
      resetForm();
    });
  }

  Future<void> pickTime() async {
    final picked = await showTimePicker(
      context: context,
      initialTime: TimeOfDay.now(),
    );
    if (picked != null) {
      setState(() => selectedTime = picked);
    }
  }

  Future<void> pickDate() async {
    final picked = await showDatePicker(
      context: context,
      initialDate: DateTime.now(),
      firstDate: DateTime(2020),
      lastDate: DateTime(2100),
    );
    if (picked != null) {
      setState(() => selectedDate = picked);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("📝 To-Do List"),
      ),
      body: SingleChildScrollView(
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 12.0, vertical: 16),
          child: Column(
            children: [
              TextField(
                controller: titleController,
                decoration: const InputDecoration(
                  labelText: 'Task Title *',
                  prefixIcon: Icon(Icons.title),
                ),
              ),
              const SizedBox(height: 12),

              TextField(
                controller: descriptionController,
                maxLines: 2,
                decoration: const InputDecoration(
                  labelText: 'Description',
                  prefixIcon: Icon(Icons.description),
                ),
              ),
              const SizedBox(height: 12),

              DropdownButtonFormField<String>(
                value: selectedPriority,
                decoration: const InputDecoration(
                  labelText: "Priority Level *",
                  prefixIcon: Icon(Icons.flag),
                ),
                items: ['Easy', 'Medium', 'High']
                    .map((level) => DropdownMenuItem(
                          value: level,
                          child: Text(level),
                        ))
                    .toList(),
                onChanged: (value) => setState(() => selectedPriority = value),
              ),
              const SizedBox(height: 12),

              ElevatedButton.icon(
                onPressed: pickDate,
                icon: const Icon(Icons.calendar_today),
                label: Text(
                  selectedDate == null
                      ? "Pick Date"
                      : "Date: ${selectedDate!.day}/${selectedDate!.month}/${selectedDate!.year}",
                ),
                style: ElevatedButton.styleFrom(
                  minimumSize: const Size.fromHeight(50),
                  backgroundColor: Colors.indigo[300],
                ),
              ),
              const SizedBox(height: 12),

              ElevatedButton.icon(
                onPressed: pickTime,
                icon: const Icon(Icons.access_time),
                label: Text(
                  selectedTime == null
                      ? "Pick Time"
                      : "Time: ${selectedTime!.format(context)}",
                ),
                style: ElevatedButton.styleFrom(
                  minimumSize: const Size.fromHeight(50),
                  backgroundColor: Colors.indigo[300],
                ),
              ),
              const SizedBox(height: 20),

              ElevatedButton.icon(
                onPressed: handleSave,
                style: ElevatedButton.styleFrom(
                  minimumSize: const Size.fromHeight(50),
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(8),
                  ),
                ),
                icon: Icon(editingIndex == null ? Icons.add : Icons.save),
                label: Text(editingIndex == null ? 'Add Task' : 'Save Changes'),
              ),
              const SizedBox(height: 20),
              const Divider(thickness: 1),
              const SizedBox(height: 10),

              if (items.isEmpty)
                const Text("No tasks added yet.", style: TextStyle(fontSize: 16))
              else
                ListView.builder(
                  shrinkWrap: true,
                  physics: const NeverScrollableScrollPhysics(),
                  itemCount: items.length,
                  itemBuilder: (context, index) {
                    final item = items[index];
                    final color = item['priority'] == 'High'
                        ? Colors.red
                        : item['priority'] == 'Medium'
                            ? Colors.orange
                            : Colors.green;

                    return Card(
                      elevation: 3,
                      margin: const EdgeInsets.symmetric(vertical: 8),
                      child: ListTile(
                        contentPadding: const EdgeInsets.symmetric(
                            horizontal: 16, vertical: 10),
                        leading: CircleAvatar(
                          backgroundColor: color.withOpacity(0.2),
                          child: Icon(Icons.flag, color: color),
                        ),
                        title: Text(
                          item['title']!,
                          style: const TextStyle(fontWeight: FontWeight.bold),
                        ),
                        subtitle: Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: [
                            if (item['description']!.isNotEmpty)
                              Padding(
                                padding: const EdgeInsets.only(top: 4.0),
                                child: Text(item['description']!),
                              ),
                            const SizedBox(height: 4),
                            Text(
                              "Priority: ${item['priority']}",
                              style: TextStyle(
                                  fontWeight: FontWeight.w500, color: color),
                            ),
                            Text("Date: ${item['date']}"),
                            Text("Time: ${item['time']}"),
                          ],
                        ),
                        trailing: Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            IconButton(
                              icon:
                                  const Icon(Icons.edit, color: Colors.indigo),
                              onPressed: () {
                                final timeParts =
                                    item['time']!.split(":").map(int.parse).toList();
                                final dateParts =
                                    item['date']!.split("/").map(int.parse).toList();

                                setState(() {
                                  titleController.text = item['title']!;
                                  descriptionController.text =
                                      item['description']!;
                                  selectedPriority = item['priority'];
                                  selectedTime = TimeOfDay(
                                    hour: timeParts[0],
                                    minute: timeParts[1],
                                  );
                                  selectedDate = DateTime(
                                    DateTime.now().year,
                                    dateParts[1],
                                    dateParts[0],
                                  );
                                  editingIndex = index;
                                });
                              },
                            ),
                            IconButton(
                              icon: const Icon(Icons.delete, color: Colors.red),
                              onPressed: () {
                                setState(() {
                                  items.removeAt(index);
                                  if (editingIndex == index) resetForm();
                                });
                              },
                            ),
                          ],
                        ),
                      ),
                    );
                  },
                ),
            ],
          ),
        ),
      ),
    );
  }
}

import 'dart:typed_data';
import 'package:flutter/material.dart';
import 'package:signature/signature.dart';

void main() {
  runApp(const CorporateSignatureApp());
}

class CorporateSignatureApp extends StatelessWidget {
  const CorporateSignatureApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Company Signature',
      theme: ThemeData(
        primarySwatch: Colors.indigo,
        useMaterial3: true,
      ),
      home: const SignatureHomePage(),
    );
  }
}

class SignatureHomePage extends StatefulWidget {
  const SignatureHomePage({super.key});

  @override
  State<SignatureHomePage> createState() => _SignatureHomePageState();
}

class _SignatureHomePageState extends State<SignatureHomePage> {
  final SignatureController _controller = SignatureController(
    penStrokeWidth: 3,
    penColor: Colors.black,
    exportBackgroundColor: Colors.transparent,
  );

  Color _currentColor = Colors.black;
  double _strokeWidth = 3;

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  Future<void> _saveSignature(BuildContext context) async {
    if (_controller.isEmpty) return;
    Uint8List? data = await _controller.toPngBytes();
    if (!context.mounted) return;
    showDialog(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text('تم إنشاء التوقيع'),
        content: data == null
            ? const Text('حدث خطأ أثناء الحفظ')
            : Image.memory(data),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('إغلاق'),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('التوقيع الإلكتروني – اسم الشركة'),
        centerTitle: true,
      ),
      body: Column(
        children: [
          // Company Header
          Container(
            width: double.infinity,
            padding: const EdgeInsets.all(12),
            color: Colors.indigo.shade50,
            child: Row(
              children: const [
                Icon(Icons.business, size: 36),
                SizedBox(width: 12),
                Text(
                  'اسم الشركة هنا',
                  style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                ),
              ],
            ),
          ),

          // Signature Area
          Expanded(
            child: Container(
              margin: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                border: Border.all(color: Colors.grey.shade400),
                borderRadius: BorderRadius.circular(12),
              ),
              child: Signature(
                controller: _controller,
                backgroundColor: Colors.white,
              ),
            ),
          ),

          // Tools
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                DropdownButton<Color>(
                  value: _currentColor,
                  items: const [
                    DropdownMenuItem(value: Colors.black, child: Text('أسود')),
                    DropdownMenuItem(value: Colors.blue, child: Text('أزرق')),
                    DropdownMenuItem(value: Colors.red, child: Text('أحمر')),
                  ],
                  onChanged: (color) {
                    if (color == null) return;
                    setState(() {
                      _currentColor = color;
                      _controller.penColor = color;
                    });
                  },
                ),
                Slider(
                  value: _strokeWidth,
                  min: 1,
                  max: 8,
                  divisions: 7,
                  label: _strokeWidth.toStringAsFixed(0),
                  onChanged: (value) {
                    setState(() {
                      _strokeWidth = value;
                      _controller.penStrokeWidth = value;
                    });
                  },
                ),
              ],
            ),
          ),

          // Actions
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: OutlinedButton.icon(
                    onPressed: _controller.clear,
                    icon: const Icon(Icons.delete_outline),
                    label: const Text('مسح'),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: () => _saveSignature(context),
                    icon: const Icon(Icons.save),
                    label: const Text('حفظ التوقيع'),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}


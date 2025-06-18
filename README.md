import 'dart:math';
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:math_expressions/math_expressions.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: FirebaseOptions(
      apiKey: 'AIzaSyAJ7GbCJwRVdRdEuiiUy_Msw_njs0YSv9w',
      appId: '1:816104024899:android:6efeb8bbfefcc040619df6',
      messagingSenderId: '816104024899',
      projectId: 'scientific-calculator-9e8e4',
      authDomain: 'scientific-calculator-9e8e4.firebaseapp.com',
    ),
  );
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(debugShowCheckedModeBanner: false, home: AuthGate());
  }
}

class AuthGate extends StatelessWidget {
  const AuthGate({super.key});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      stream: FirebaseAuth.instance.authStateChanges(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return Scaffold(body: Center(child: CircularProgressIndicator()));
        }
        if (snapshot.hasData) {
          return Calculator();
        }
        return SignInPage();
      },
    );
  }
}

class SignInPage extends StatefulWidget {
  const SignInPage({super.key});

  @override
  _SignInPageState createState() => _SignInPageState();
}

class _SignInPageState extends State<SignInPage> {
  final emailController = TextEditingController();
  final passwordController = TextEditingController();
  String error = '';
  bool isLoading = false;

  Future<void> signIn() async {
    setState(() {
      error = '';
      isLoading = true;
    });
    try {
      await FirebaseAuth.instance.signInWithEmailAndPassword(
        email: emailController.text.trim(),
        password: passwordController.text,
      );
    } on FirebaseAuthException catch (e) {
      setState(() => error = 'Login failed: ${e.message}');
    } catch (e) {
      setState(() => error = 'Login failed: $e');
    } finally {
      setState(() => isLoading = false);
    }
  }

  void goToSignUp() {
    Navigator.push(context, MaterialPageRoute(builder: (_) => SignUpPage()));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      body: Padding(
        padding: const EdgeInsets.all(24.0),
        child: Center(
          child: SingleChildScrollView(
            child: Column(
              children: [
                Text(
                  'Sign In',
                  style: TextStyle(color: Colors.white, fontSize: 28),
                ),
                SizedBox(height: 20),
                TextField(
                  controller: emailController,
                  decoration: inputDecoration('Email'),
                  style: TextStyle(color: Colors.white),
                ),
                SizedBox(height: 10),
                TextField(
                  controller: passwordController,
                  decoration: inputDecoration('Password'),
                  obscureText: true,
                  style: TextStyle(color: Colors.white),
                ),
                SizedBox(height: 20),
                isLoading
                    ? CircularProgressIndicator()
                    : ElevatedButton(onPressed: signIn, child: Text('Sign In')),
                TextButton(
                  onPressed: goToSignUp,
                  child: Text(
                    "Don't have an account? Sign Up",
                    style: TextStyle(color: Colors.blueAccent),
                  ),
                ),
                SizedBox(height: 12),
                Text(error, style: TextStyle(color: Colors.redAccent)),
              ],
            ),
          ),
        ),
      ),
    );
  }

  InputDecoration inputDecoration(String hint) {
    return InputDecoration(
      hintText: hint,
      filled: true,
      fillColor: Colors.white24,
      border: OutlineInputBorder(),
    );
  }
}

class SignUpPage extends StatefulWidget {
  const SignUpPage({super.key});

  @override
  State<SignUpPage> createState() => _SignUpPageState();
}

class _SignUpPageState extends State<SignUpPage> {
  final emailController = TextEditingController();
  final passwordController = TextEditingController();
  String error = '';
  bool isLoading = false;

  bool isPasswordValid(String password) {
    final regex = RegExp(
      r'^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[!@#\$&*~]).{8,}$',
    );
    return regex.hasMatch(password);
  }

  Future<void> signUp() async {
    setState(() {
      error = '';
      isLoading = true;
    });

    final email = emailController.text.trim();
    final password = passwordController.text;

    if (email.isEmpty || password.isEmpty) {
      setState(() {
        error = 'Email and Password cannot be empty.';
        isLoading = false;
      });
      return;
    }

    if (!isPasswordValid(password)) {
      setState(() {
        error =
            'Password must be at least 8 characters and include uppercase, lowercase, number, and special character.';
        isLoading = false;
      });
      return;
    }

    try {
      await FirebaseAuth.instance.createUserWithEmailAndPassword(
        email: email,
        password: password,
      );
      Navigator.pop(context); // Go back to Sign In page on success
    } on FirebaseAuthException catch (e) {
      setState(() {
        error = 'Sign-up failed: ${e.message}';
      });
    } catch (e) {
      setState(() {
        error = 'Sign-up failed: ${e.toString()}';
      });
    } finally {
      setState(() {
        isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(title: Text("Sign Up")),
      body: Padding(
        padding: const EdgeInsets.all(24.0),
        child: Center(
          child: SingleChildScrollView(
            child: Column(
              children: [
                TextField(
                  controller: emailController,
                  decoration: inputDecoration('Email'),
                  style: TextStyle(color: Colors.white),
                ),
                SizedBox(height: 10),
                TextField(
                  controller: passwordController,
                  decoration: inputDecoration('Password'),
                  obscureText: true,
                  style: TextStyle(color: Colors.white),
                ),
                SizedBox(height: 20),
                isLoading
                    ? CircularProgressIndicator()
                    : ElevatedButton(
                      onPressed: signUp,
                      child: Text('Create Account'),
                    ),
                SizedBox(height: 12),
                Text(error, style: TextStyle(color: Colors.redAccent)),
              ],
            ),
          ),
        ),
      ),
    );
  }

  InputDecoration inputDecoration(String hint) {
    return InputDecoration(
      hintText: hint,
      filled: true,
      fillColor: Colors.white24,
      border: OutlineInputBorder(),
    );
  }
}

class Calculator extends StatefulWidget {
  const Calculator({super.key});

  @override
  _CalculatorState createState() => _CalculatorState();
}

class _CalculatorState extends State<Calculator> {
  String expression = '';
  String result = '';
  bool isDegree = true;

  final List<String> buttons = [
    'Deg/Rad',
    'C',
    '⌫',
    '(',
    ')',
    'sin',
    'cos',
    'tan',
    'log',
    'ln',
    'sin⁻¹',
    'cos⁻¹',
    'tan⁻¹',
    'x²',
    'x³',
    'nPr',
    'nCr',
    '!',
    '√',
    'x^n',
    'π',
    'e',
    'rnd',
    'root',
    '%',
    '7',
    '8',
    '9',
    '/',
    '^',
    '4',
    '5',
    '6',
    '*',
    '',
    '1',
    '2',
    '3',
    '-',
    '',
    '0',
    '.',
    '=',
    '+',
    '',
  ];

  void onButtonPressed(String text) {
    setState(() {
      switch (text) {
        case 'C':
          expression = '';
          result = '';
          break;
        case '⌫':
          if (expression.isNotEmpty) {
            expression = expression.substring(0, expression.length - 1);
          }
          break;
        case '=':
          evaluate();
          break;
        case '√':
          expression += 'sqrt(';
          break;
        case 'log':
        case 'ln':
        case 'sin':
        case 'cos':
        case 'tan':
          expression += '$text(';
          break;
        case 'sin⁻¹':
          expression += 'asin(';
          break;
        case 'cos⁻¹':
          expression += 'acos(';
          break;
        case 'tan⁻¹':
          expression += 'atan(';
          break;
        case '!':
          expression += '!';
          break;
        case 'x²':
          expression += '^2';
          break;
        case 'x³':
          expression += '^3';
          break;
        case 'x^n':
          expression += '^';
          break;
        case 'π':
          expression += 'pi';
          break;
        case 'e':
          expression += 'e';
          break;
        case '%':
          expression += '/100';
          break;
        case 'nPr':
          expression += 'nPr';
          break;
        case 'nCr':
          expression += 'nCr';
          break;
        case 'rnd':
          expression += 'rnd';
          break;
        case 'root':
          expression += 'root(';
          break;
        case 'Deg/Rad':
          isDegree = !isDegree;
          break;
        case '(':
        case ')':
          expression += text;
          break;
        case '':
          break;
        default:
          expression += text;
      }
    });
  }

  double factorial(int n) {
    if (n < 0) throw Exception("Negative factorial");
    double res = 1;
    for (int i = 1; i <= n; i++) {
      res *= i;
    }
    return res;
  }

  double nCr(int n, int r) {
    if (r > n) return 0;
    return factorial(n) / (factorial(r) * factorial(n - r));
  }

  double nPr(int n, int r) {
    if (r > n) return 0;
    return factorial(n) / factorial(n - r);
  }

  double nthRoot(double base, double n) {
    return pow(base, 1 / n).toDouble();
  }

  String preprocess(String exp) {
    // Replace π and rnd
    exp = exp.replaceAll('π', 'pi');
    exp = exp.replaceAll('rnd', Random().nextDouble().toString());

    // Convert degrees to radians for trig functions if needed
    if (isDegree) {
      exp = exp.replaceAllMapped(
        RegExp(r'(sin|cos|tan|asin|acos|atan)\(([^()]+)\)'),
        (m) => '${m[1]}(((${m[2]}) * pi / 180))',
      );
    }

    // Replace factorial: e.g. 5! with its value
    exp = exp.replaceAllMapped(RegExp(r'(\d+)!'), (m) {
      int n = int.parse(m[1]!);
      return factorial(n).toString();
    });

    // Replace nCr: e.g. 5nCr3
    exp = exp.replaceAllMapped(RegExp(r'(\d+)nCr(\d+)'), (m) {
      int n = int.parse(m[1]!);
      int r = int.parse(m[2]!);
      return nCr(n, r).toString();
    });

    // Replace nPr: e.g. 5nPr3
    exp = exp.replaceAllMapped(RegExp(r'(\d+)nPr(\d+)'), (m) {
      int n = int.parse(m[1]!);
      int r = int.parse(m[2]!);
      return nPr(n, r).toString();
    });

    // Replace root(x, n) with pow(x, 1/n)
    exp = exp.replaceAllMapped(RegExp(r'root\(([^,]+),([^()]+)\)'), (m) {
      double base = double.parse(m[1]!.trim());
      double root = double.parse(m[2]!.trim());
      return pow(base, 1 / root).toString();
    });

    // Replace log(x, base) with log(x)/log(base)
    exp = exp.replaceAllMapped(RegExp(r'log\(([^,]+),([^()]+)\)'), (m) {
      return '(log(${m[1]})/log(${m[2]}))';
    });

    // Replace ln(x) with log(x)/log(e)
    exp = exp.replaceAllMapped(RegExp(r'ln\(([^)]+)\)'), (m) {
      return '(log(${m[1]})/log(e))';
    });

    return exp;
  }

  void evaluate() async {
    final invalidEnd = RegExp(r'[+\-*/^(\.]$');
    if (expression.trim().isEmpty || invalidEnd.hasMatch(expression)) {
      setState(() => result = 'Invalid Expression');
      return;
    }

    try {
      final processed = preprocess(expression);

      Parser p = Parser();
      Expression exp = p.parse(processed);
      ContextModel cm = ContextModel();
      cm.bindVariableName('pi', Number(pi));
      cm.bindVariableName('e', Number(e));

      final evalResult = exp.evaluate(EvaluationType.REAL, cm);
      setState(() {
        result = evalResult.toString();
      });

      await FirebaseFirestore.instance.collection('history').add({
        'expression': expression,
        'result': result,
        'timestamp': FieldValue.serverTimestamp(),
      });
    } catch (e) {
      setState(() {
        result = 'Error';
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(title: Text('Scientific Calculator')),
      body: Column(
        children: [
          Container(
            padding: EdgeInsets.all(12),
            alignment: Alignment.centerRight,
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.end,
              children: [
                Text(
                  'Mode: ${isDegree ? 'Degrees' : 'Radians'}',
                  style: TextStyle(color: Colors.orange, fontSize: 14),
                ),
                SizedBox(height: 4),
                Text(
                  expression,
                  style: TextStyle(fontSize: 18, color: Colors.white70),
                ),
                Text(
                  result,
                  style: TextStyle(fontSize: 22, color: Colors.white),
                ),
              ],
            ),
          ),
          Divider(color: Colors.white30),
          Expanded(
            flex: 2,
            child: GridView.builder(
              padding: EdgeInsets.all(6),
              itemCount: buttons.length,
              gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 5,
                crossAxisSpacing: 4,
                mainAxisSpacing: 4,
              ),
              itemBuilder: (context, index) {
                String btn = buttons[index];
                return ElevatedButton(
                  style: ElevatedButton.styleFrom(
                    padding: EdgeInsets.zero,
                    backgroundColor:
                        btn == '='
                            ? Colors.orange
                            : (btn == 'C' || btn == '⌫')
                            ? Colors.red
                            : Colors.grey[850],
                    shape: RoundedRectangleBorder(
                      borderRadius: BorderRadius.circular(6),
                    ),
                  ),
                  onPressed: btn.isEmpty ? null : () => onButtonPressed(btn),
                  child: Center(
                    child: Text(
                      btn,
                      style: TextStyle(fontSize: 14, color: Colors.white),
                    ),
                  ),
                );
              },
            ),
          ),
          Divider(color: Colors.white24),
          Expanded(
            flex: 1,
            child: StreamBuilder<QuerySnapshot>(
              stream:
                  FirebaseFirestore.instance
                      .collection('history')
                      .orderBy('timestamp', descending: true)
                      .limit(10)
                      .snapshots(),
              builder: (context, snapshot) {
                if (!snapshot.hasData) {
                  return Center(child: CircularProgressIndicator());
                }
                final docs = snapshot.data!.docs;
                return ListView.builder(
                  itemCount: docs.length,
                  itemBuilder: (context, index) {
                    final data = docs[index];
                    return ListTile(
                      title: Text(
                        '${data['expression']} = ${data['result']}',
                        style: TextStyle(color: Colors.white70),
                      ),
                    );
                  },
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

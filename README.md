import 'package:flutter/material.dart';

void main() {
  runApp(const AgenciaViagensApp());
}

class AgenciaViagensApp extends StatelessWidget {
  const AgenciaViagensApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Agência de Viagens',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        brightness: Brightness.light,
        appBarTheme: const AppBarTheme(
          backgroundColor: Colors.indigo,
          titleTextStyle: TextStyle(
            color: Colors.white,
            fontSize: 22,
            fontWeight: FontWeight.bold,
          ),
        ),
        floatingActionButtonTheme: const FloatingActionButtonThemeData(
          backgroundColor: Colors.indigo,
        ),
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final List<Review> _reviews = [];
  String _searchQuery = '';

  void _addReview(String comment, int rating) {
    setState(() {
      _reviews.add(Review(comment: comment, rating: rating));
    });
  }

  void _removeReview(int index) {
    setState(() {
      _reviews.removeAt(index);
    });
  }

  void _updateSearchQuery(String query) {
    setState(() {
      _searchQuery = query.toLowerCase();
    });
  }

  List<Review> get _filteredReviews {
    if (_searchQuery.isEmpty) {
      return _reviews;
    }
    return _reviews.where((review) {
      return review.comment.toLowerCase().contains(_searchQuery);
    }).toList();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Avaliações de Viagens'),
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: TextField(
              onChanged: _updateSearchQuery,
              decoration: InputDecoration(
                prefixIcon: const Icon(Icons.search),
                hintText: 'Pesquisar avaliações...',
                filled: true,
                fillColor: Colors.grey[200],
                border: OutlineInputBorder(
                  borderRadius: BorderRadius.circular(10.0),
                  borderSide: BorderSide.none,
                ),
              ),
            ),
          ),
          Expanded(
            child: ReviewList(
              reviews: _filteredReviews,
              onRemoveReview: _removeReview,
            ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _showAddReviewDialog,
        child: const Icon(Icons.add),
      ),
    );
  }

  void _showAddReviewDialog() {
    showDialog(
      context: context,
      builder: (context) {
        return AddReviewDialog(onAddReview: _addReview);
      },
    );
  }
}

class Review {
  final String comment;
  final int rating;

  Review({required this.comment, required this.rating});
}

class ReviewList extends StatelessWidget {
  final List<Review> reviews;
  final Function(int) onRemoveReview;

  const ReviewList({
    super.key,
    required this.reviews,
    required this.onRemoveReview,
  });

  @override
  Widget build(BuildContext context) {
    if (reviews.isEmpty) {
      return const Center(
        child: Text(
          'Nenhuma avaliação encontrada.',
          style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
        ),
      );
    }
    return ListView.builder(
      itemCount: reviews.length,
      itemBuilder: (context, index) {
        final review = reviews[index];
        return Dismissible(
          key: Key(review.comment),
          background: Container(color: Colors.red),
          onDismissed: (direction) {
            onRemoveReview(index);
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text('Avaliação removida')),
            );
          },
          child: ReviewCard(
            review: review,
          ),
        );
      },
    );
  }
}

class ReviewCard extends StatelessWidget {
  final Review review;

  const ReviewCard({super.key, required this.review});

  @override
  Widget build(BuildContext context) {
    return Card(
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(15.0),
      ),
      margin: const EdgeInsets.symmetric(vertical: 8, horizontal: 16),
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              review.comment,
              style: const TextStyle(
                fontSize: 16,
                fontWeight: FontWeight.bold,
              ),
            ),
            const SizedBox(height: 8),
            Row(
              children: List.generate(review.rating, (index) {
                return const Icon(Icons.star, color: Colors.amber);
              }),
            ),
            const SizedBox(height: 8),
            ElevatedButton.icon(
              onPressed: () {},
              icon: const Icon(Icons.thumb_up),
              label: const Text('Curtir'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.indigoAccent,
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(10),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class AddReviewDialog extends StatefulWidget {
  final Function(String, int) onAddReview;

  const AddReviewDialog({super.key, required this.onAddReview});

  @override
  _AddReviewDialogState createState() => _AddReviewDialogState();
}

class _AddReviewDialogState extends State<AddReviewDialog> {
  final _commentController = TextEditingController();
  int _rating = 1;

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(15)),
      title: const Text('Adicionar Avaliação'),
      content: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          TextField(
            controller: _commentController,
            decoration: InputDecoration(
              hintText: 'Digite seu comentário',
              filled: true,
              fillColor: Colors.grey[200],
              border: OutlineInputBorder(
                borderRadius: BorderRadius.circular(10.0),
                borderSide: BorderSide.none,
              ),
            ),
          ),
          const SizedBox(height: 10),
          const Text('Avaliação:'),
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: List.generate(5, (index) {
              return IconButton(
                icon: Icon(
                  index < _rating ? Icons.star : Icons.star_border,
                  color: Colors.amber,
                ),
                onPressed: () {
                  setState(() {
                    _rating = index + 1;
                  });
                },
              );
            }),
          ),
        ],
      ),
      actions: [
        TextButton(
          onPressed: () {
            final comment = _commentController.text;
            if (comment.isNotEmpty) {
              widget.onAddReview(comment, _rating);
              Navigator.of(context).pop();
            }
          },
          child: const Text('Adicionar'),
        ),
      ],
    );
  }
}

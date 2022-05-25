struct line {
int length;
char contents[0];
};

struct line *thisline = (struct line *)
	malloc (sizeof (struct line) + this_length);
thisline->length = this_length;
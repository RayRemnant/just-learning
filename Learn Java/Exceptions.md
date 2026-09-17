unchecked expetions 
not required to write code to handle these exceptions
// can happen, but it's a bug

checked exceptions
compiler forces you to handle these expetions
usually means you can't prevent a certain flow from happening > need to handle it

class Throwable 
	Exception
		RuntimeException (unchecked) 
	Error (unchecked)
		StackOverflowError
		OutOfMemoryError
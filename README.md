# twilio
Twilio C++11/14 API 
by Nikita Chumakov &lt;nikkikom@gmail.com&gt;

Simplifies the generating TwiML rules, can be used in http servers. The simple
server (based on cpp-netlib) provided in tests directory.

API is header-only library. The classes resides in "twilio::ml" namespace. We
are going to use this namespace in the rest of the code: "using namespace
twilio::ml;".

The generating of ML responses can be acompletished with std::ostream-s ans very
simple:

  using namespace twilio::ml;
  std::cout << (
    Response () 
      << Play ("http://domain.com/smth.mp3")
      << Say ("Hello")
  ) << "\n";

Output:
  <Response>
  <Play>http://domain.com/smth.mp3</Play>
  <Say>Hello</Say>
  </Response>

Please not that parenthesis before Response() call and after Say(...) must
always be used due to operators precedence rules in C++.

Nested verbs can be generated with the use of parenthesis. Also I implemented
named arguments for verbs calls:

  Response () <<
    ( Gather::setTimeout<6> ("action")
      << Say::setLoop<10>
            ::setVoice <Say::woman> 
            ("Please leave a message after the tone.")
      << Pause::setLength<10> ()
      << Play::setLoop<2> ("http://com/m.mp3")
    )
    << Say ("hi");

Output:
  <Response>
  <Gather action="action" timeout="6">
  <Say loop="woman" loop="10">Please leave a message after the tone.</Say>
  <Pause length="10"/>
  <Play loop="2">http://com/m.mp3</Play>
  </Gather>
  <Say>hi</Say>
  </Response>

The API checks verb's nesting rules and if you try to nest incompatible vers the library will generate the descriptive error:

  std::cout << (
    Response () 
      << ( Play ("http://domain.com/smth.mp3") << Say ("Hello") )
  ) << "\n";


In file included from runtime.cc:1:
../include/twilio/ml.h:96:2: error: static_assert failed "Nesting rules
      violation: Inner cannot be nested under Outer"
        static_assert (verbs::is_nestable<Outer, Inner>::value,
        ^              ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
runtime.cc:12:44: note: in instantiation of function template specialization
      'twilio::ml::operator<<<twilio::ml::verbs::Play<twilio::nargs::field<unsigned
      int, 1> >, twilio::ml::verbs::Say<twilio::nargs::field<unsigned int, 1>,
      twilio::nargs::field<int, 5>, twilio::nargs::field<int, 3> >, void, void,
      void>' requested here
                << ( Play ("http://domain.com/smth.mp3") << Say ("Hello") )
                                                         ^
1 error generated. 


The API can generate data structure in constant context (during compile time):

  constexpr auto response = Response () <<
    ( Gather::setTimeout<6> ("action")
      << Say::setLoop<10>
            ::setVoice <Say::woman> 
            ("Please leave a message after the tone.")
      << Pause::setLength<10> ()
      << Play::setLoop<2> ("http://com/m.mp3")
    )
    << Say ("hi");

    std::cout << response << '\n';

Due to compiler limitations is not supported on clang in c++11 mode. 
Clang c++14, g++ c++11, g++ c++14 work fine. Clang c++11 works fine in runtime
context.

Compilers tested: 
  gcc 4.9.2 (c++11 and c++14 modes)
  gcc 5.0.0 20150308
  clang 3.6.0
  
How to compile the tests:
  c++ -std=c++14 -I ../include -o constexpr constexpr.cc -Wall
  c++ -std=c++11 -I ../include -o runtime runtime.cc -Wall 

http servers test example needs the recent boost and cpp-netlib header only 
library (http://cpp-netlib.org/).

  c++ -std=c++14 -DBOOST_NETWORK_NO_LIB -I ../../cpp-netlib/ -I ../include \
    -o http_server http_server.cc \
    -lboost_system-mt -lboost_thread-mt -lssl -lcrypto

Bugs and limitations:

Due to a lack of time (had only one day to work with library) only a small 
subset of verbs is implemented. Basically it is not more than proof of concept 
that such syntax is possbile and that the generation of data structures in
constant context is possible.

http_server returns only one predefined response, while it should check the
query and return different responses.

The way I pass nouns into the vers should probably reworked. It is difficult to
pass string literals by named arguments (in current implementation), so I pass
them with constructors. However this is not very elegant and should be changed.


### What is a Monad?

Now, we are in a position to explain what a Monad is.

Looking back at the four examples, what did they have in common? In all
four cases, We had a type constructor with one type parameter - `IO`,
`Maybe`, `Either String` and `Writer` all take a type parameter.

And, for all four of these examples, we had a bind function. For `IO`,
we had the `\>\>=` function and for the others we had the bind functions
that we wrote ourselves.

``` {.haskell}
bindWriter :: Writer a -> (a -> Writer b) -> Writer b
bindEither :: Either String a -> (a -> Either String b) -> Either String b
bindMaybe :: Maybe a -> (a -> Maybe b) -> Maybe b
```

How the bind works depends on the case. In the case of `IO` it is
built-in magic, but you can think of it as just combining the two plans
describing the actions to take during computation. For `bindMaybe` and
`bindEither` the logic is for the whole plan to fail if any part of it
fails, and for `bindWriter`, the logic was to combine the list of log
messages.

And that is the main idea of Monads. It\'s a concept of computation with
some additional side effects, and the ability to bind two such
computations together.

There is another aspect that we briefly mentioned in the case of `IO`
but not for the other examples - another thing that we can always do.

Whenever we have such a concept of computation with side effects, we
also also always have the ability to produce a computation of this kind
that `doesn\'t` have any side effects.

In the example of `IO`, this was done with `return`. Given an `a`, you
can create an `IO a` which is the recipe that always simply returns the
`a` with no side effects. Each of the other example has this ability as
well, as shown below.

``` {.haskell}
return              :: a -> IO a
Just                :: a -> Maybe a
Right               :: a -> Either String a
(\a -> Writer a []) :: a -> Writer a
```

And it is the combination of these two features that defines a Monad.

-   the ability to bind two computations together
-   the possibility to construct a computation from a pure value without
    making use of any of the potential side effects

If we look in the REPL:

``` {.haskell}
Prelude Week04.Contract> :i Monad
type Monad :: (` -> `) -> Constraint
class Applicative m => Monad m where
(>>=) :: m a -> (a -> m b) -> m b
(>>) :: m a -> m b -> m b
return :: a -> m a
{-# MINIMAL (>>=) #-}
   -- Defined in ‘GHC.Base’
instance Monad (Either e) -- Defined in ‘Data.Either’
instance Monad [] -- Defined in ‘GHC.Base’
instance Monad Maybe -- Defined in ‘GHC.Base’
instance Monad IO -- Defined in ‘GHC.Base’
instance Monad ((->) r) -- Defined in ‘GHC.Base’
instance (Monoid a, Monoid b, Monoid c) => Monad ((,,,) a b c)
-- Defined in ‘GHC.Base’
instance (Monoid a, Monoid b) => Monad ((,,) a b)
-- Defined in ‘GHC.Base’
instance Monoid a => Monad ((,) a) -- Defined in ‘GHC.Base’
```

We see the bind function

``` {.haskell}
(>>=) :: m a -> (a -> m b) -> m b
```

And the `return` function that takes a pure value and turns it into a
computation that has potential for side effects, but does not use them.

``` {.haskell}
return :: a -> m a
```

The other function `\>\>` can easily be defined in terms of `\>\>=`, but
is provided for convenience.

``` {.haskell}
(>>) :: m a -> m b -> m b
```

What this function does is to throw away the result of the first
computation, so you could define it in terms of `\>\>=` by just ignoring
the argument to the function parameter.

There\'s another technical computation. We see that `Monad` has the
super class `Applicative`, so every Monad is `Applicative`.

``` {.haskell}
Prelude Week04.Contract> :i Applicative
type Applicative :: (` -> `) -> Constraint
class Functor f => Applicative f where
pure :: a -> f a
(<`>) :: f (a -> b) -> f a -> f b
GHC.Base.liftA2 :: (a -> b -> c) -> f a -> f b -> f c
(`>) :: f a -> f b -> f b
(<`) :: f a -> f b -> f a
{-# MINIMAL pure, ((<`>) | liftA2) #-}
   -- Defined in ‘GHC.Base’
instance Applicative (Either e) -- Defined in ‘Data.Either’
instance Applicative [] -- Defined in ‘GHC.Base’
instance Applicative Maybe -- Defined in ‘GHC.Base’
instance Applicative IO -- Defined in ‘GHC.Base’
instance Applicative ((->) r) -- Defined in ‘GHC.Base’
instance (Monoid a, Monoid b, Monoid c) =>
         Applicative ((,,,) a b c)
-- Defined in ‘GHC.Base’
instance (Monoid a, Monoid b) => Applicative ((,,) a b)
-- Defined in ‘GHC.Base’
instance Monoid a => Applicative ((,) a) -- Defined in ‘GHC.Base’
```

We see it has a bunch of functions, but we only need the first two.

``` {.haskell}
pure :: a -> f a
(<`>) :: f (a -> b) -> f a -> f b
```

The function `pure` has the same type signature as `return`. Then there
is \<\`\> (pronounced \'ap\') which looks a bit more complicated. But,
the truth is that, once you have `return` and `\>\>=` in a Monad, we can
easily define both `pure` and \<\`\>.

We see that `Applicative` also has a superclass `Functor`.

``` {.haskell}
Prelude Week04.Contract> :i Functor
type Functor :: (` -> `) -> Constraint
class Functor f where
fmap :: (a -> b) -> f a -> f b
(<$) :: a -> f b -> f a
{-# MINIMAL fmap #-}
   -- Defined in ‘GHC.Base’
instance Functor (Either a) -- Defined in ‘Data.Either’
instance Functor [] -- Defined in ‘GHC.Base’
instance Functor Maybe -- Defined in ‘GHC.Base’
instance Functor IO -- Defined in ‘GHC.Base’
instance Functor ((->) r) -- Defined in ‘GHC.Base’
instance Functor ((,,,) a b c) -- Defined in ‘GHC.Base’
instance Functor ((,,) a b) -- Defined in ‘GHC.Base’
instance Functor ((,) a) -- Defined in ‘GHC.Base’
```

As we mentioned in the context of `IO`, `Functor` has the `fmap`
function which, given a function from `a` to `b` will turn an `f a` into
an `f b`.

The prototypical example for `fmap` is lists where `fmap` is just `map`.
Given a function from `a` to `b`, you can create a list of type `b` from
a list of type `a` by applying the `map` function to each of the
elements of the list.

Again, once you have `return` and `\>\>=`, it is easy to define `fmap`.

So, whenever you want to define a Monad, you just define `return` and
`\>\>=`, and to make the compiler happy and to give instances for
`Functor` and `Applicative`, there\'s always a standard way of doing it.

We can do this in the example of `Writer`.

``` {.haskell}
import Control.Monad

instance Functor Writer where
   fmap = liftM

instance Applicative Writer where
   pure = return
   (<`>) = ap

instance Monad Writer where
   return a = Writer a []
   (>>=) = bindWriter
```

We don\'t have to do the same for `Maybe`, `Either` or `IO` because they
are already Monads defined by the Prelude.

### Why Is This useful?

It is always useful, in general, to identify a common pattern and give
it a name.

But, maybe the most important advantage is that there are lots of
functions that don\'t care which Monad we are dealing with - they will
work with all Monads.

Let\'s generalize the example where we compute the sum of three
integers. We use a `let` in the example below for reasons that will
become clear in moment.

``` {.haskell}
threeInts :: Monad m => m Int -> m Int -> m Int -> m Int
threeInts mx my mz =
   mx >>= \k ->
   my >>= \l ->
   mz >>= \m ->
   let s = k + l + m in return s
```

Now we have this function, we can return to the `Maybe` example and
rewrite it.

``` {.haskell}
foo'' :: String -> String -> String -> Maybe Int
foo'' x y z = threeInts (readMaybe x) (readMaybe y) (readMaybe z)
```

We can do the same for the `Either` example.

``` {.haskell}
foo'' :: String -> String -> String -> Either String Int
foo'' x y z = threeInts (readEither x) (readEither y) (readEither z)
```

The `Writer` example is not exactly the same.

If we are happy not to have the log message for the sum, it is very
simple as it is already an instance of `threeInts`.

``` {.haskell}
foo'' :: Writer Int -> Writer Int -> Writer Int -> Writer Int
foo'' x y z = threeInts
```

However, if we want the final log message, it becomes a little more
complicated.

``` {.haskell}
foo'' :: Writer Int -> Writer Int -> Writer Int -> Writer Int
foo'' x y z = do
   s <- threeInts x y z
   tell ["sum: " ++ show s]
   return s
```

If you look into the Control.Monad module in the standard Haskell
Prelude, you will see that there are many useful functions that you can
use for all Monads.

One way to think about a Monad is as a computation with a super power.

In the case of `IO`, the super power would be having real-world
side-effects. In the case of `Maybe`, the super power is being able to
fail. The super power of `Either` is to fail with an error message. And
in the case of `Writer`, the super power is to log messages.

There is a saying in the Haskell community that Haskell has an
overloaded semi-colon. The explanation for this is that in many
imperative programming languages, you have semi-colons that end with a
semi-colon - each statement is executed one after the other, each
separated by a semi-colon. But, what exactly the semi-colon means
depends on the language. For example, there could be an exception, in
which case computation would stop and wouldn\'t continue with the next
lines.

In a sense, `bind` is like a semi-colon. And the cool thing about
Haskell is that it is a programmable semi-colon. We get to say what the
logic is for combining two computations together.

Each Monad comes with its own \"semi-colon\".

### \'do\' notation

Because this pattern is so common and monadic computations are all over
the place, there is a special notation for this in Haskell, called `do`
notation.

It is syntactic sugar. Let\'s rewrite `threeInts` using `do` notation.

``` {.haskell}
threeInts' :: Monad m => m Int -> m Int -> m Int -> m Int
threeInts' mx my mz = do
   k <- mx
   l <- my
   m <- mz
   let s = k + l + m
   return s
```

This does exactly the same thing as the non-`do` version, but it has
less noise.

Note that the `let` statement does not use an `in` part. It does not
need to inside a `do` block.

And that\'s Monads. There is a lot more to say about them but hopefully
you now have an idea of what Monads are and how they work.

Often you are in a situation where you want several effects at once -for
example you may want optional failure `and` log messages. There are ways
to do that in Haskell. For example there are Monad Transformers where
one can basically build custom Monads with the features that you want.

There are other approaches. One is called Effect Systems, which has a
similar objective. And this is incidentally what Plutus uses for
important Monads. In particular the Contact Monad in the wallet, and the
Trace Monad which is used to test Plutus code.

The good news is that you don\'t need to understand Effect Systems to
work with these Monads. You just need to know that you are working with
a Monad, and what super powers it has.

Plutus Monads
-------------

Now that we have seen how to write monadic code, either by using bind
and return or by using do notation, we can look a very important Monad,
namely the Contract Monad, which you may have already noticed in
previous code examples.

The Contract Monad defines code that will run in the wallet, which is
the off-chain part of Plutus.

But, before we go into details, we will talk about a second Monad, the
EmulatorTrace monad.

### The EmulatorTrace Monad

You may have wondered if there is a way to execute Plutus code for
testing purposes without using the Plutus Playground. There is indeed,
and this is done using the `EmulatorTrace` Monad.

You can think of a program in this monad as what we do manually in the
`simulator` tab of the playground. That is, we define the initial
conditions, we define the actions such as which wallets invoke which
endpoints with which parameters and we define the waiting periods
between actions.

The relevant definitions are in the package `plutus-contract` in module
`Plutus.Trace.Emulator`.

``` {.haskell}
module Plutus.Trace.Emulator
```

The most basic function is called `runEmulatorTrace`.

``` {.haskell}
-- | Run an emulator trace to completion, returning a tuple of the final state
-- of the emulator, the events, and any error, if any.
runEmulatorTrace
    :: EmulatorConfig
    -> EmulatorTrace ()
    -> ([EmulatorEvent], Maybe EmulatorErr, EmulatorState)
runEmulatorTrace cfg trace =
    (\(xs :> (y, z)) -> (xs, y, z))
    $ run
    $ runReader ((initialDist . _initialChainState) cfg)
    $ foldEmulatorStreamM (generalize list)
    $ runEmulatorStream cfg trace
```

It gets something called an `EmulatorConfig` and an `EmulatorTrace ()`,
which is a pure computation where no real-world side effects are
involved. It is a pure function that executes the trace on an emulated
blockchain, and then gives a result as a list of `EmulatorEvent`s, maybe
an error, if there was one, and then finally the final `EmulatorState`.

`EmulatorConfig` is defined in a different module in the same package:

``` {.haskell}
module Wallet.Emulator.Stream

data EmulatorConfig =
EmulatorConfig
    { _initialChainState      :: InitialChainState -- ^ State of the blockchain at the beginning of the simulation. Can be given as a map of funds to wallets, or as a block of transactions.
    } deriving (Eq, Show)

type InitialChainState = Either InitialDistribution Block
```

We see it only has one field, which is of type `InitialChainState` and
it is either `InitialDistribution` or `Block`.

`InitialDistribution` is defined in another module in the same package,
and it is a type synonym for a map of key value pairs of `Wallet`s to
`Value`s, as you would expect. `Value` can be either lovelace or native
tokens.

``` {.haskell}
module Plutus.Contract.Trace

type InitialDistribution = Map Wallet Value
```

In the same module, we see something called `defaultDist` which returns
a default distribution for all wallets. It does this by passing the 10
wallets defined by `allWallets` to `defaultDistFor` which takes a list
of wallets.

``` {.haskell}
-- | The wallets used in mockchain simulations by default. There are
--   ten wallets because the emulator comes with ten private keys.
allWallets :: [EM.Wallet]
allWallets = EM.Wallet <$> [1 .. 10]

defaultDist :: InitialDistribution
defaultDist = defaultDistFor allWallets

defaultDistFor :: [EM.Wallet] -> InitialDistribution
defaultDistFor wallets = Map.fromList $ zip wallets (repeat (Ada.lovelaceValueOf 100_000_000))
```

We can try this out in the REPL:

``` {.haskell}
Prelude Week04.Contract> import Plutus.Trace.Emulator
Prelude Plutus.Trace.Emulator Week04.Contract> import Plutus.Contract.Trace
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Week04.Contract> defaultDist
fromList [(Wallet 1,Value (Map [(,Map [("",100000000)])])),(Wallet 2,Value (Map [(,Map [("",100000000)])])),(Wallet 3,Value (Map [(,Map [("",100000000)])])),(Wallet 4,Value (Map [(,Map [("",100000000)])])),(Wallet 5,Value (Map [(,Map [("",100000000)])])),(Wallet 6,Value (Map [(,Map [("",100000000)])])),(Wallet 7,Value (Map [(,Map [("",100000000)])])),(Wallet 8,Value (Map [(,Map [("",100000000)])])),(Wallet 9,Value (Map [(,Map [("",100000000)])])),(Wallet 10,Value (Map [(,Map [("",100000000)])]))]
```

We can see that each of the 10 wallets has been given an initial
distribution of 100,000,000 lovelace.

We can also get the balances for a specific wallet or wallets:

``` {.haskell}
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Week04.Contract> defaultDistFor [Wallet 1]
fromList [(Wallet 1,Value (Map [(,Map [("",100000000)])]))]
```

If you want different initial values, of if you want native tokens, then
you have to specify that manually.

Let\'s see what we need to run our first trace:

``` {.haskell}
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Week04.Contract> :t runEmulatorTrace
runEmulatorTrace
:: EmulatorConfig
-> EmulatorTrace ()
-> ([Wallet.Emulator.MultiAgent.EmulatorEvent], Maybe EmulatorErr,
      Wallet.Emulator.MultiAgent.EmulatorState)
```

So, we need an `EmulatorConfig` which we know takes an
`InitialChainState`.

``` {.haskell}
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Week04.Contract> import Wallet.Emulator.Stream 
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Wallet.Emulator.Stream Week04.Contract> :i InitialChainState 
type InitialChainState :: `
type InitialChainState =
Either InitialDistribution Ledger.Blockchain.Block
      -- Defined in ‘Wallet.Emulator.Stream’
```

If we take the `Left` of the `defaultDist` will will get an
`InitialDistribution`.

``` {.haskell}
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Wallet.Emulator.Stream Week04.Contract> :t Left defaultDist
Left defaultDist :: Either InitialDistribution b
```

Which we can then use to construct an `EmulatorConfig`.

``` {.haskell}
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Wallet.Emulator.Stream Week04.Contract> EmulatorConfig $ Left defaultDist
EmulatorConfig {_initialChainState = Left (fromList [(Wallet 1,Value (Map [(,Map [("",100000000)])])),(Wallet 2,Value (Map [(,Map [("",100000000)])])),(Wallet 3,Value (Map [(,Map [("",100000000)])])),(Wallet 4,Value (Map [(,Map [("",100000000)])])),(Wallet 5,Value (Map [(,Map [("",100000000)])])),(Wallet 6,Value (Map [(,Map [("",100000000)])])),(Wallet 7,Value (Map [(,Map [("",100000000)])])),(Wallet 8,Value (Map [(,Map [("",100000000)])])),(Wallet 9,Value (Map [(,Map [("",100000000)])])),(Wallet 10,Value (Map [(,Map [("",100000000)])]))])}
```

So, let\'s try out `runEmulatorTrace`. Recall that, as well as and
`EmulatorConfig`, we also need to pass in an `EmulatorTrace`, and the
most simple one we can create is simply one that returns Unit - `return
()`.

``` {.haskell}
runEmulatorTrace (EmulatorConfig $ Left defaultDist) $ return ()
```

If you run this in the REPL you will get a crazy amount of data output
to the console, even though we are not doing anything with the trace. If
you want to make it useful, you must somehow filter all this data down
to something that sensible, and aggregate it in some way.

Luckily, there are other functions as well as `runEmulatorTrace`. One of
them is `runEmulatorTraceIo` which runs the emulation then outputs the
trace in a nice form on the screen.

``` {.haskell}
runEmulatorTraceIO
:: EmulatorTrace ()
-> IO ()
runEmulatorTraceIO = runEmulatorTraceIO' def def
```

To use this function, we don\'t need to specify an `EmulatorConfig` like
we did before, because by default will will just use the default
distribution.

In the REPL:

``` {.haskell}
Prelude...> runEmulatorTraceIO $ return ()
```

``` {.}
Slot 00000: TxnValidate af5e6d25b5ecb26185289a03d50786b7ac4425b21849143ed7e18bcd70dc4db8
Slot 00000: SlotAdd Slot 1
Slot 00001: SlotAdd Slot 2
Final balances
Wallet 1: 
{, ""}: 100000000
Wallet 2: 
{, ""}: 100000000
Wallet 3: 
{, ""}: 100000000
Wallet 4: 
{, ""}: 100000000
Wallet 5: 
{, ""}: 100000000
Wallet 6: 
{, ""}: 100000000
Wallet 7: 
{, ""}: 100000000
Wallet 8: 
{, ""}: 100000000
Wallet 9: 
{, ""}: 100000000
Wallet 10: 
{, ""}: 100000000
```

And we see a much more manageable, concise output. Nothing happens, but
we see the Genesis transaction and then the final balances for each
wallet.

If you want more control, there is also `runEmulatorTraceIO\'`, which
does take an `EmulatorConfig`, so we could specify a different
distribution.

``` {.haskell}
runEmulatorTraceIO'
:: TraceConfig
-> EmulatorConfig
-> EmulatorTrace ()
-> IO ()
runEmulatorTraceIO' tcfg cfg trace
= runPrintEffect (outputHandle tcfg) $ runEmulatorTraceEff tcfg cfg trace
```

It also takes a `TraceConfig`, which has two fields.

``` {.haskell}
data TraceConfig = TraceConfig
{ showEvent    :: EmulatorEvent' -> Maybe String
-- ^ Function to decide how to print the particular events.
, outputHandle :: Handle
-- ^ Where to print the outputs to. Default: 'System.IO.stdout'
}
```

The first field, `showEvent` is a function that specifies which
`EmulatorEvent`s are displayed and how they are displayed. It takes an
`EmulatorEvent` as an argument and can return `Nothing` it the event
should not be displayed, or a `Just` with a `String` showing how the
event will be displayed.

Here is the default `TraceConfig` used by `runEmulatorTraceIO`. We can
see that most events are ignored and that we only get output for some of
the events.

``` {.haskell}
instance Default TraceConfig where
def = TraceConfig
            { showEvent     = defaultShowEvent
            , outputHandle  = stdout
            }

defaultShowEvent :: EmulatorEvent' -> Maybe String
defaultShowEvent = \case
UserThreadEvent (UserLog msg)                                        -> Just $ "``` USER LOG: " <> msg
InstanceEvent (ContractInstanceLog (ContractLog (A.String msg)) _ _) -> Just $ "``` CONTRACT LOG: " <> show msg
InstanceEvent (ContractInstanceLog (StoppedWithError err)       _ _) -> Just $ "``` CONTRACT STOPPED WITH ERROR: " <> show err
InstanceEvent (ContractInstanceLog NoRequestsHandled            _ _) -> Nothing
InstanceEvent (ContractInstanceLog (HandledRequest _)           _ _) -> Nothing
InstanceEvent (ContractInstanceLog (CurrentRequests _)          _ _) -> Nothing
SchedulerEvent _                                                     -> Nothing
ChainIndexEvent _ _                                                  -> Nothing
WalletEvent _ _                                                      -> Nothing
ev                                                                   -> Just . renderString . layoutPretty defaultLayoutOptions . pretty $ ev
```

The second field is a handle which defaults to `stdout`, but we could
also specify a file here.

Now let\'s look at a more interesting trace, using the `Vesting`
contract from the last lecture.

First, we define a `Trace`.

``` {.haskell}
myTrace :: EmulatorTrace ()
myTrace = do
h1 <- activateContractWallet (Wallet 1) endpoints
h2 <- activateContractWallet (Wallet 2) endpoints
callEndpoint @"give" h1 $ GiveParams
      { gpBeneficiary = pubKeyHash $ walletPubKey $ Wallet 2
      , gpDeadline    = Slot 20
      , gpAmount      = 1000
      }
void $ waitUntilSlot 20
callEndpoint @"grab" h2 ()
void $ waitNSlots 1
```

The first thing we have to do is to activate the wallets using the
monadic function `activateContractWallet`. We bind the result of this
function to `h1`, and then bind the result of a second call (for Wallet
2) to `h2`. Those two values - `h1` and `h2` are handles to their
respective wallets.

Next, we use `callEndpoint` to simulate Wallet 1 calling the `give`
endpoint, with the shown parameters. We then wait for 20 slots. The
function `waitUntilSlot` actually returns a value representing the slot
that was reached, but, as we are not interested in that value here, we
use `void` to ignore it. We then simulate the call to the `grab`
endpoint by Wallet 2.

Now, we can write a function to call `runEmulatorTraceIO` with out
`Trace`.

``` {.haskell}
test :: IO ()
test = runEmulatorTraceIO myTrace
```

And, we can then run this in the REPL:

``` {.haskell}
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Wallet.Emulator Week04.Trace Wallet.Emulator.Stream Week04.Contract> test
```

``` {.}
Slot 00000: TxnValidate af5e6d25b5ecb26185289a03d50786b7ac4425b21849143ed7e18bcd70dc4db8
Slot 00000: SlotAdd Slot 1
Slot 00001: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
  Contract instance started
Slot 00001: 00000000-0000-4000-8000-000000000001 {Contract instance for wallet 2}:
  Contract instance started
Slot 00001: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
  Receive endpoint call: Object (fromList [("tag",String "give"),("value",Object (fromList [("unEndpointValue",Object (fromList [("gpAmount",Number 1000.0),("gpBeneficiary",Object (fromList [("getPubKeyHash",String "39f713d0a644253f04529421b9f51b9b08979d08295959c4f3990ee617f5139f")])),("gpDeadline",Object (fromList [("getSlot",Number 20.0)]))]))]))])
Slot 00001: W1: TxSubmit: 49f326a21c09ba52eddee46b65bdb5fb33b3444745e9af1510a68f9043696eba
Slot 00001: TxnValidate 49f326a21c09ba52eddee46b65bdb5fb33b3444745e9af1510a68f9043696eba
Slot 00001: SlotAdd Slot 2
Slot 00002: ``` CONTRACT LOG: "made a gift of 1000 lovelace to 39f713d0a644253f04529421b9f51b9b08979d08295959c4f3990ee617f5139f with deadline Slot {getSlot = 20}"
Slot 00002: SlotAdd Slot 3
Slot 00003: SlotAdd Slot 4
Slot 00004: SlotAdd Slot 5
Slot 00005: SlotAdd Slot 6
Slot 00006: SlotAdd Slot 7
Slot 00007: SlotAdd Slot 8
Slot 00008: SlotAdd Slot 9
Slot 00009: SlotAdd Slot 10
Slot 00010: SlotAdd Slot 11
Slot 00011: SlotAdd Slot 12
Slot 00012: SlotAdd Slot 13
Slot 00013: SlotAdd Slot 14
Slot 00014: SlotAdd Slot 15
Slot 00015: SlotAdd Slot 16
Slot 00016: SlotAdd Slot 17
Slot 00017: SlotAdd Slot 18
Slot 00018: SlotAdd Slot 19
Slot 00019: SlotAdd Slot 20
Slot 00020: 00000000-0000-4000-8000-000000000001 {Contract instance for wallet 2}:
  Receive endpoint call: Object (fromList [("tag",String "grab"),("value",Object (fromList [("unEndpointValue",Array [])]))])
Slot 00020: W2: TxSubmit: d9a2028384b4472242371f27cb51727f5c7c04327972e4278d1f69f606019a8b
Slot 00020: TxnValidate d9a2028384b4472242371f27cb51727f5c7c04327972e4278d1f69f606019a8b
Slot 00020: SlotAdd Slot 21
Slot 00021: ``` CONTRACT LOG: "collected gifts"
Slot 00021: SlotAdd Slot 22
Final balances
Wallet 1: 
    {, ""}: 99998990
Wallet 2: 
    {, ""}: 100000990
Wallet 3: 
    {, ""}: 100000000
Wallet 4: 
    {, ""}: 100000000
Wallet 5: 
    {, ""}: 100000000
Wallet 6: 
    {, ""}: 100000000
Wallet 7: 
    {, ""}: 100000000
Wallet 8: 
    {, ""}: 100000000
Wallet 9: 
    {, ""}: 100000000
Wallet 10: 
    {, ""}: 100000000
```

This output is very similar to the output we see in the playground. We
can see the Genesis transaction as well as both the `give` and `grab`
transactions from the `Trace`. We can also see some log output from the
contract itself, prefixed with `CONTRACT LOG`.

We can also log from inside the `Trace` monad. We could, for example,
lof the result of the final `waitNSlots` call:

``` {.haskell}
myTrace :: EmulatorTrace ()
myTrace = do
...
...
s <- waitNSlots 1
Extras.logInfo $ "reached slot " ++ show s
```

We would then see this output when we run the emulation:

``` {.}
...
Slot 00020: SlotAdd Slot 21
Slot 00021: ``` USER LOG: reached slot Slot {getSlot = 21}
Slot 00021: ``` CONTRACT LOG: "collected gifts"
Slot 00021: SlotAdd Slot 22
...
```

Now let\'s look at the Contract Monad.

### The Contract Monad

The purpose of the Contract Monad is to define off-chain code that runs
in the wallet. It has four type parameters:

``` {.haskell}
newtype Contract w s e a = Contract { unContract :: Eff (ContractEffs w s e) a }
      deriving newtype (Functor, Applicative, Monad)
```

The `a` is the same as in every Monad - it denotes the result type of
the computation.

We will go into the other three in more detail later but just briefly:

-   w is like our Writer monad example, it allows us to write log
    messages of type `w`.
-   s describes the blockchain capabilities, e.g. waiting for a slot,
    submitting transactions, getting the wallet\'s public key. It can
    also contain specific endpoints.
-   e describes the type of error messages that this monad can throw.

Let\'s write an example.

``` {.haskell}
myContract1 :: Contract () BlockchainActions Text ()
myContract1 = Contract.logInfo @String "Hello from the contract!"
```

Here, we pass a `Contract` constructed with `Unit` as the `w` type and
`BlockchainActions` as the second argument, `s`. This gives us access to
all the blockchain actions - the only thing we can\'t do is to call
specific endpoints.

For `e` - the error message type, we use `Text`. `Text` is a Haskell
type which is like `String`, but it is much more efficient.

We don\'t want a specific result, so we use `Unit` for the type `a`.

For the function body, we write a log message. We use `\@String`
because, we have imported the type `Data.Text` and we have used the
`OverloadedStrings` GHC compiler option, so the compiler needs to know
what type we are referencing - a `Text` or a `String`. We can use
`\@String` if we also use the compiler option `TypeApplications`.

Let\'s now define a `Trace` that starts the contract in the wallet, and
a `test` function to run it.

``` {.haskell}
myTrace1 :: EmulatorTrace ()
myTrace1 = void $ activateContractWallet (Wallet 1) myContract1

test1 :: IO ()
test1 = runEmulatorTraceIO myTrace1
```

If we run this in the REPL, we will see our log message from the
contract.

``` {.Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Wallet.Emulator Week04.Trace Wallet.Emulator.Stream Week04.Contract> test1
Slot 00000: TxnValidate af5e6d25b5ecb26185289a03d50786b7ac4425b21849143ed7e18bcd70dc4db8
Slot 00000: SlotAdd Slot 1
Slot 00001: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
Contract instance started
Slot 00001: ``` CONTRACT LOG: \"Hello from the contract!\"
Slot 00001: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
Contract instance stopped (no errors)
Slot 00001: SlotAdd Slot 2
Final balances
Wallet 1: 
{, \"\"}: 100000000
Wallet 2: 
{, \"\"}: 100000000
Wallet 3: 
{, \"\"}: 100000000
Wallet 4: 
{, \"\"}: 100000000
Wallet 5: 
{, \"\"}: 100000000
Wallet 6: 
{, \"\"}: 100000000
Wallet 7: 
{, \"\"}: 100000000
Wallet 8: 
{, \"\"}: 100000000
Wallet 9: 
{, \"\"}: 100000000
Wallet 10: 
{, \"\"}: 100000000}
```

Now, let\'s throw an exception.

``` {.haskell}
myContract1 :: Contract () BlockchainActions Text ()
myContract1 = do
void $ Contract.throwError "BOOM!"
Contract.logInfo @String "Hello from the contract!"
```

Recall that we chose the type `Text` as the error message.

``` {.}
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Wallet.Emulator Week04.Trace Wallet.Emulator.Stream Week04.Contract> test1
Slot 00000: TxnValidate af5e6d25b5ecb26185289a03d50786b7ac4425b21849143ed7e18bcd70dc4db8
Slot 00000: SlotAdd Slot 1
Slot 00001: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
Contract instance started
Slot 00001: ``` CONTRACT STOPPED WITH ERROR: "\"BOOM!\""
Slot 00001: SlotAdd Slot 2
Final balances
Wallet 1: 
{, ""}: 100000000
Wallet 2: 
{, ""}: 100000000
Wallet 3: 
{, ""}: 100000000
Wallet 4: 
{, ""}: 100000000
Wallet 5: 
{, ""}: 100000000
Wallet 6: 
{, ""}: 100000000
Wallet 7: 
{, ""}: 100000000
Wallet 8: 
{, ""}: 100000000
Wallet 9: 
{, ""}: 100000000
Wallet 10: 
{, ""}: 100000000
```

Now, we don\'t get the log message, but we do get told that the contract
stopped with an error and we see our exception message.

Another thing you can do is to handle exceptions. We will use the
`handleError` function from module `Plutus.Contract.Types`.

``` {.haskell}
handleError ::
      forall w s e e' a.
      (e -> Contract w s e' a)
      -> Contract w s e a
      -> Contract w s e' a
handleError f (Contract c) = Contract c' where
      c' = E.handleError @e (raiseUnderN @'[E.Error e'] c) (fmap unContract f)
```

The `handleError` function takes an error handler and a `Contract`
instance. The error handler takes an argument of type `e` from our
contract, and returns a new `Contract` with the same type parameters as
the first, but we can change the type of the `e` argument - the error
type, which is expressed in the return `Contract` argument list as
`e\'`.

``` {.haskell}
myContract2 :: Contract () BlockchainActions Void ()
myContract2 = Contract.handleError
      (\err -> Contract.logError $ "Caught error: " ++ unpack err)
      myContract1

myTrace2 :: EmulatorTrace ()
myTrace2 = void $ activateContractWallet (Wallet 1) myContract2

test2 :: IO ()
test2 = runEmulatorTraceIO myTrace2
```

We use the type `Void` as the error type. `Void` is a type that can hold
no value, so, by using this type we are saying that there cannot be any
errors for this contract.

::: {.note}
::: {.title}
Note
:::

The function `unpack` is defined in the `Data.Text` module. It converts
a value of type `Text` to a value of type `String`.
:::

``` {.}
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Wallet.Emulator Week04.Trace Wallet.Emulator.Stream Week04.Contract> test2
Slot 00000: TxnValidate af5e6d25b5ecb26185289a03d50786b7ac4425b21849143ed7e18bcd70dc4db8
Slot 00000: SlotAdd Slot 1
Slot 00001: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
Contract instance started
Slot 00001: ``` CONTRACT LOG: "Caught error: BOOM!"
Slot 00001: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
Contract instance stopped (no errors)
Slot 00001: SlotAdd Slot 2
Final balances
...
```

We no longer get the error message, but, instead we get a message from
the error handler showing the exception that was thrown by Contract1.
Note that we still do not get the message \"Hello from the contract!\".
Contract 1 still stopped processing after its error, but there was no
overall contract error due to the exception being caught and handled.

Of course, exceptions can also happen even if they are not explicitly
thrown by your contract code. There are operations, such as submitting a
transaction where there are insufficient inputs to make a payment for an
output, where Plutus will throw an exception.

Next, let\'s look at the `s` parameter, the second parameter to
`Contract`, that determines the available blockchain actions.

In the first two examples we just used the `BlockChainActions` type
which has all the standard functionality but without support for
specific endpoints. If we want support for specific endpoints, we must
use a different type.

The way that is usually done is by using a type synonym. The following
example will create a type synonym `MySchema` that has all the
capabilities of `BlockChainActions` but with the addition of being able
to call endpoint `foo` with an argument of type `Int`.

``` {.haskell}
type MySchema = BlockchainActions .\/ Endpoint "foo" Int
```

::: {.note}
::: {.title}
Note
:::

The operator `.\\/` is a type operator - it operates on types, not
values. In order to use this we need to use the `TypeOperators` and
`DataKinds` compiler options.
:::

Now, we can use the `MySchema` type to define our contract.

``` {.haskell}
myContract3 :: Contract () MySchema Text ()
myContract3 = do
      n <- endpoint @"foo"
      Contract.logInfo n
```

This contract will block until the endpoint `foo` is called with, in our
case, an `Int`. Then the value of the `Int` parameter will be bound to
`n`. Because of this, it is no longer enough for us to just activate the
contract to test it. Now, we must invoke the endpoint as well.

In order to do this, we now need to handle from
`activateContractWallet`, which we can then use to call the endpoint.

``` {.haskell}
myTrace3 :: EmulatorTrace ()
myTrace3 = do
      h <- activateContractWallet (Wallet 1) myContract3
      callEndpoint @"foo" h 42

test3 :: IO ()
test3 = runEmulatorTraceIO myTrace3
```

Running this in the REPL:

``` {.}
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Wallet.Emulator Week04.Trace Wallet.Emulator.Stream Week04.Contract> test3
Slot 00000: TxnValidate af5e6d25b5ecb26185289a03d50786b7ac4425b21849143ed7e18bcd70dc4db8
...
Receive endpoint call: Object (fromList [("tag",String "foo"),("value",Object (fromList [("unEndpointValue",Number 42.0)]))])
Slot 00001: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
Contract log: Number 42.0
...
Final balances
...
Wallet 10: 
{, ""}: 100000000
```

Finally, let\'s look at the first type parameter, the writer. The `w`
cannot be an arbitrary type, it must be an instance of the type class
`Monoid`.

``` {.haskell}
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Wallet.Emulator Week04.Trace Wallet.Emulator.Stream Week04.Contract> :i Monoid
type Monoid :: ` -> Constraint
class Semigroup a => Monoid a where
mempty :: a
mappend :: a -> a -> a
mconcat :: [a] -> a
{-# MINIMAL mempty #-}
      -- Defined in ‘GHC.Base’
instance Monoid [a] -- Defined in ‘GHC.Base’
instance Monoid Ordering -- Defined in ‘GHC.Base’
instance Semigroup a => Monoid (Maybe a) -- Defined in ‘GHC.Base’
instance Monoid a => Monoid (IO a) -- Defined in ‘GHC.Base’
instance Monoid b => Monoid (a -> b) -- Defined in ‘GHC.Base’
instance (Monoid a, Monoid b, Monoid c, Monoid d, Monoid e) =>
      Monoid (a, b, c, d, e)
-- Defined in ‘GHC.Base’
instance (Monoid a, Monoid b, Monoid c, Monoid d) =>
      Monoid (a, b, c, d)
-- Defined in ‘GHC.Base’
instance (Monoid a, Monoid b, Monoid c) => Monoid (a, b, c)
-- Defined in ‘GHC.Base’
instance (Monoid a, Monoid b) => Monoid (a, b)
-- Defined in ‘GHC.Base’
instance Monoid () -- Defined in ‘GHC.Base’
```

This is a very important and very common type class in Haskell. It
defines `mempty` and `mappend`.

The function `mempty` is like the neutral element, and `mappend`
combines two elements of this type to create a new element of the same
type.

The prime example of a `Monoid` is `List`, when `mempty` is the empty
list `\[\]`, and `mappend` is concatenation `++`.

For example:

``` {.haskell}
Prelude> mempty :: [Int]
[]
Prelude> mappend [1, 2, 3 :: Int] [4, 5, 6]
[1,2,3,4,5,6]
```

The are many, many other examples of the `Monoid` type, and we will see
other instances in this course.

But for now, let\'s stick with lists and write our last example.

``` {.haskell}
myContract4 :: Contract [Int] BlockchainActions Text ()
myContract4 = do
    void $ Contract.waitNSlots 10
    tell [1]
    void $ Contract.waitNSlots 10
    tell [2]
    void $ Contract.waitNSlots 10
```

Rather than using `Unit` as our `w` type, we are using `\[Int\]`. This
allows us to use the `tell` function as shown.

This now gives us access to those messages during the trace, using the
`observableState` function.

``` {.haskell}
myTrace4 :: EmulatorTrace ()
myTrace4 = do
    h <- activateContractWallet (Wallet 1) myContract4

    void $ Emulator.waitNSlots 5
    xs <- observableState h
    Extras.logInfo $ show xs

    void $ Emulator.waitNSlots 10
    ys <- observableState h
    Extras.logInfo $ show ys

    void $ Emulator.waitNSlots 10
    zs <- observableState h
    Extras.logInfo $ show zs

test4 :: IO ()
test4 = runEmulatorTraceIO myTrace4
```

If we run this in the REPL, we can see the `USER LOG` messages created
using the `tell` function.

``` {.}
Prelude Plutus.Trace.Emulator Plutus.Contract.Trace Wallet.Emulator Week04.Trace Wallet.Emulator.Stream Week04.Contract> test4
Slot 00000: TxnValidate af5e6d25b5ecb26185289a03d50786b7ac4425b21849143ed7e18bcd70dc4db8
Slot 00000: SlotAdd Slot 1
Slot 00001: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
  Contract instance started
Slot 00001: SlotAdd Slot 2
...
Slot 00005: SlotAdd Slot 6
Slot 00006: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
  Sending contract state to Thread 0
Slot 00006: SlotAdd Slot 7
Slot 00007: ``` USER LOG: []
Slot 00007: SlotAdd Slot 8
...
Slot 00015: SlotAdd Slot 16
Slot 00016: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
  Sending contract state to Thread 0
Slot 00016: SlotAdd Slot 17
Slot 00017: ``` USER LOG: [1]
Slot 00017: SlotAdd Slot 18
...
Slot 00025: SlotAdd Slot 26
Slot 00026: 00000000-0000-4000-8000-000000000000 {Contract instance for wallet 1}:
  Sending contract state to Thread 0
Slot 00026: SlotAdd Slot 27
Slot 00027: ``` USER LOG: [1,2]
Final balances
Wallet 1: 
    {, ""}: 100000000
Wallet 2: 
    {, ""}: 100000000
...
Wallet 10: 
    {, ""}: 100000000
```

Using this mechanism, it is possible to pass information from the
contract running in the wallet to the outside world. Using endpoints we
can pass information into a contract. And using the `tell` mechanism we
can get information out of the wallet.

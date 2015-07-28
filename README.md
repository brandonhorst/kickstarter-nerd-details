# Lacona Kickstarter Nerd Details

Technical information about developing custom Lacona Commands. For the [Kickstarter Campaign](https://www.kickstarter.com/projects/2102999333/lacona-natural-language-commands-for-your-mac). Read at your own risk.

Lacona extentions are written in Javascript, which run in a Node.js process alongside the app. They have the full power of Node.js along with their disposal, along with an additional custom Lacona library for interfacing with some Mac-specific features like Spotlight. Commands can also make use of AppleScript to interact with other apps.

Lacona supports two different kinds of addons - **commands**, which give Lacona new functionality, and **extensions**, which build upon existing functionality. Both commands and extensions are defined using Javascript code. At launch, you won't be able to extend Lacona without touching some code. At some point, I would love to develop a GUI command and extension development, but that would be a stretch goal.

## Commands

I'll explain by way of example. You could make a new command that allows users to tweet. You define two things: the **grammar** (sentence structure) and the **logic** (the code that physically sends the tweet to Twitter). The image below shows the grammar in a pseudocode format. To see what the code will really look like, check out the [show me the code](#show-me-the-code) section.

![Command Structure](https://raw.github.com/lacona/kickstarter-nerd-details/master/images/TweetCommand@2x.png)

That's it - you just specify the sentence structure, and assign an `id` to the relevant pieces. You don't need to think about the parsing logic at all.

Lacona users who install your command can now make some insightful tweets.

![Command Structure](https://raw.github.com/lacona/kickstarter-nerd-details/master/images/TweetSentence@2x.png)

A few things you should notice:

- You do not need to do anything fancy to allow the user to input a string that is less than 140 characters - that is all built into Lacona with the `<String>` component.
- You are providing many different ways for the user to input the same thing. This flexibility allows anyone to use Lacona in the way that is most natural for them.
- Someone can translate your command, and as long as you they put in a `<String>` with `id = message`, it will work with the same code. Here is an example with Spanish.

![Command Structure](https://raw.github.com/lacona/kickstarter-nerd-details/master/images/TweetCommand 1@2x.png)

Notice that we have added a new language, with a new grammar, but the code hasn't changed at all. You can do this for every language under the sun.

![Command Structure](https://raw.github.com/lacona/kickstarter-nerd-details/master/images/SpanishSentence@2x.png)

You can think of commands as a list of additional sentences that Lacona will be able to understand. They each have a single verb, and they do a single thing.

## Extensions

Extensions build on top of commands. If commands define new sentences, then Extensions provide new phrases with which you can compose those sentences.

Let's have another example. It may be useful to post the same thing to different social networks. Let's say we want the user to be able enter `tweet my last Facebook status` - a useful thing for those who use multiple social networks.

![Command Structure](https://raw.github.com/lacona/kickstarter-nerd-details/master/images/TweetSentence 1@2x.png)

We already have a Tweet command, but right now, entering `tweet my last Facebook status` would not have the desired results. It would interpret `my last Facebook status` as a string, and post it literally.

![Command Structure](https://raw.github.com/lacona/kickstarter-nerd-details/master/images/TweetSentence@2x.png)

We don't want to rewrite our Tweet Command - we should keep it as general as possible. After all, we wouldn't want to modify the command to fit into every other social network. Instead, we can just extend the `<String>` component.

![Command Structure](https://raw.github.com/lacona/kickstarter-nerd-details/master/images/Extension@2x.png)

Now, any time a `<String>` is used in any Lacona command, `my last Facebook status` will also be an option. Of course, it is possible that the user may want to tweet that exact message, so both options will be presented side-by-side.

![Command Structure](https://raw.github.com/lacona/kickstarter-nerd-details/master/images/TweetSentence 2@2x.png)

If the user selects the italicized, purple option, it will have the desired effect.

![Command Structure](https://raw.github.com/lacona/kickstarter-nerd-details/master/images/TwitterShot 1@2x.png)

Many of the built-in commands use the `<String>` component, so the user will now be able to use their last Facebook status in other contexts as well. This may or may not be useful, but the option is always available.

![Command Structure](https://raw.github.com/lacona/kickstarter-nerd-details/master/images/SearchSentence@2x.png)

# Show me the Code

All Lacona commands run on Node.js. While they can be written in plain JS, the preferred way to write them is actually to use [JSX](https://facebook.github.io/react/docs/jsx-in-depth.html) - a superset of Javascript developed by Facebook for use with the React user interface library. Grammars are defined declaratively using an XML-like syntax, which allows for greater clarity for deeply-nested structures. The code is then transpiled into pure JS using [Babel](https://babeljs.io/) This all may change a bit before the final API release, but it will give you a general idea. Of course, the final API release will include lots of pretty documentation.

Here is an example of the Tweet command that we created above, in code.

```js
/** @jsx createElement */
import {createElement, Phrase} from 'lacona-phrase'

class TweetContent extends Phrase {
  describe () {
    return (
      <argument text='message'>
        <String maxlen={140} />
      </argument>
    )
  }
}

class TweetCommand extends Phrase {
  describe () {
    return (
      <choice>
        <sequence>
          <literal text='tweet ' />
          <TweetContent id='message' />
        </sequence>
        <sequence>
          <literal text='post ' />
          <TweetContent id='message' />
          <literal text='to twitter' />
        </sequence>
        <sequence>
          <literal text='post to Twitter ' />
          <TweetContent id='message' />
        </sequence>
      </choice>
    )
  }

  execute (result) {
    send_to_twitter(result.message)
  }
}
```

And here is an example of the `my last Facebook status` extension.

```js
/** @jsx createElement */
import {createElement, Phrase} from 'lacona-phrase'
import String from 'LaconaCommand-String'

class FacebookExtension extends Phrase {
  getValue () {
    return get_last_facebook_status()
  }

  describe () {
    return <literal text='my last Facebook status' category='symbol' />
  }
}

FacebookExtension.extends = [String]
```

## Internationalization

Of course, the problem with Natural Language interfaces is that they're fairly difficult to translate. It's not enough to simply translate the labels in a form and call it a day. Here are just a few common design issues.

- Languages have different word order, including the position of verbs
- Some languages have accented/modified characters, which can be essential but are a pain to type
- Some languages have dramatic differences between dialects
- Some languages have multiple sets of characters with the same meaning
- Some languages separate words with spaces, others do not
- Some languages are written left-to-right, some right-to-left
- In some languages, it is common to use foreign words or characters to express certain concepts even if words to represent them do exist in the language
- In some languages, the user enters text in one character set, and replaced with different characters over time
- Some languages have words that simply do not have an equivalent in other languages
- Some languages use distinguish between different verbs depending upon the verb's object, while other languages do not
- Clearly, there are a lot of issues. Because of this, Lacona's language processing is as general as possible. It does not rely on the verb coming at the beginning of a sentence, it does not rely on words being separated by spaces, and it does not rely on any particular character set. It takes any input, interprets it according to its Commands, forms an data structure based upon the input, and sends the data to the command to execute.

This means that the *language* of Commands can be translated, while keeping the *code* the same. Developers do not need to know any other languages, or anything about language processing at all. And translators don't need to know anything about code. Lacona handles all of that.

## Under the Hood

Lacona does not know English. That is to say, it cannot break a sentence into grammatical components, and it cannot infer any meaning from an arbitrary sentence. It understands only the sentences that have been described for it in Commands, and nothing more.

This means that everything that Lacona can do must be translated manually into every new language that it is to support. Lacona will only launch with support for US and UK English. However, all of the Lacona commands will be open-source, so that multilingual users from around the world can add new translations.

To get a bit more technical, you can think of Lacona Commands as closer Formal Grammars, or very glorified Regular Expressions. It takes arbitrary strings as input, and interprets them according to its rules. I am not an expert in computational linguistics, but I can provide a bit more information. the biggest differences are:

- Lacona Commands are not necessarily Regular or Context-Free. Commands are composed of components, which can be thought of as similar to rules in a formal grammar. However complex components can be much more complex - they can even make use of arbitrary code to handle possible inputs.
- Lacona Commands are designed to not only interpret an string after it has been input, but also while it is being input. For example, the string "617-" is very clearly not a valid Phone Number, but it is possible that it is the beginning of a Phone Number. Lacona needs to identify that.
- Lacona command components can be evaluated dynamically, so grammars can change state based on external factors, such as the contents of the clipboard, results of scripts or network requests, or the current date.
- Altogether, this means that Lacona has a language processing system that is general enough to support all of the quirks of language, but simple enough be accessible. Simple commands are easy, and very complex commands are possible.

## Further information

Of course, this is just an introduction. Much more information will be available with the release of the API. If you have further questions, please ask on [Kickstarter](https://www.kickstarter.com/projects/2102999333/lacona-natural-language-commands-for-your-mac) or [Twitter](https://twitter.com/lacona).

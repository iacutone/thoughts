---
title: "My Elixir Conf 2023 Experience"
date: 2023-09-17T12:37:58-04:00
draft: false
toc: false
images:
tags: 
  - elixir
---


My Call For Proposal was accepted for this year's ElixirConf. This was the first time one of my CFP's was accepted to any conference so I was both very excited and very nervoud by the prospect to speak. After some thought, I accepted the speaking invitation. Now, I had around six weeks to prepare for the conference and it was an adventure.

The title of my presentation is “Learn Stochastic Gradient Descent in Under 30 Minutes.” For some background, I am a web application developer and don’t have much experience in the realm of machine learning. I took this as an opportunity to take a deep dive into the mechanics of how stochastic gradient descent works.

In order to prepare for the presentation, I had three primary sources including video series by Jeremy Howard (fast.ai) and Andrej Karpathy (micrograd) and later, a blog post by Sean Moriarty. At first, my focus and intent was to re-write Karpathy’s micrograd framework in an Elixir LiveBook. From the experience of transferring the Python micrograd code to Elixir, I came to realize that it would be difficult to explain the concepts. The micrograd framework depends on a tree data structure and in order to update tree nodes, one must use recursion.

My head does not understand recursion well. I struggled with getting the LiveBook code to work correctly. It was good to have the original micrograd Jupyter notebook answers to verify that my code was working correctly. After a long battle, I was able to get the first part of my micrograd in Elixir to work as intended. 

More issues arose when trying to implement the neural network portion of micrograd. Creating the tree node structures correctly with a singular output layer was challenging to re-write from the original Python. After spending far too much time trying to get the neural network to perform correctly, I realized that if I am struggling so much porting this code to Elixir, it won’t be possible to explain what the neural network is doing to a wider audience.

I had taken the fast.ai course numerous times throughout the years. I highly recommend this course for anyone interested in learning how to apply deep neural network concepts. Instead of trying to re-implement the neural network from micrograd, I decided to switch tactics and have the second portion of my presentation focus on how stochastic gradient descent can update weights and biases in order to solve for a function. In the fasi.ai, there is an example Kaggle notebook which uses stochastic gradient descent to solve for a parabolic function. 

I was trying to have my presentation reach as wide an audience as possible. So instead of solving for a parabolic function, I re-used the Kaggle Jupyter notebook to solve for a linear function. A linear function is the easiest concept one can use to apply stochastic gradient descent.

I combined the fast.ai Kaggle notebook with a blog post by Sean Moriarty in order to describe stochastic gradient descent in Elixir. Once I had the LiveBook code in a working state, I began writing slides to compliment the LiveBook. In hindsight, I think it would have been easier to begin by writing the words of the presentation before creating the slides. I found it difficult and time consuming to keep the presentation and slides in phase.

At cars.com, we had a group of ElixirConf presenters. Before the coference, we had several meetings with the presenters to hone our presentations which I found invaluable. The feedback from the first practice runs of my presentation was that the code samples were confusing. 

One suggestion that helped immensely is the concept of a [code sandwich](https://bphogan.com/2020/02/05/improve-tutorials-with-code-sandwich/). This concept has three parts:

A brief high-level description of the code.
The code itself.
A deeper dive into the tricky bits of the code.
I took a complementary view of this concept and instead of diving into tricky bits of code, I reinforced concepts via graphs and animations. Iterating on a solid base presentation I was able to mitigate confusion. By the third practice run, I was confident in both my slides, words and presentation as a whole.

A week before the talk I faced some severe stress. At cars.com we have an on-call rotation. While I enjoy the change of pace and different problems I face being on-call, hurricane Idalia also forced me to evacuate my apartment the same week. Faced with both on-call responsibilities and evacuating St. Petersburg with my family left little time to focus on presentation preparation. It was during this week when the voice in my head started having thoughts about formulating an excuse to not go to Orlando.

I decided to travel to Orlando early to prepare. The JW Marriott Orlando Bonnet Creek Resort & Spa hosted ElixirConf in 2023. This was a wonderful hotel and venue. But, the hotel is kind of an oasis surrounded by freeways. It was impossible to go anywhere outside of the hotel without a taxi service.

In preparation, I memorized the speech from a Google doc. During the memorization process, I transferred the Google doc to paper. By writing the words on paper, I found some words that could be adjusted and made clearer. I found the process of writing the speech on paper helped me memorize the words.

I think three days was a sufficient amount of time to memorize a twenty minute presentation. The day before I gave the presentation, I went through the entire thing in a Zoom call with a product manager at cars.com. The act of having an audience, one last time, increased my confidence even more.

I was scheduled to speak on the first day of talks at 11:30 in the morning. On the day of my talk, I skipped all of the morning talks and went for a jog in the balmy Orlando heat. While jogging, I went through the speech in my head one final time. I believe the jog helped squelch the anxiety I was about to face when giving the speech.

The cars.com crew was very supportive and showed up early to my presentation. They all sat in the first row which was awesome. The lights were extremely bright on the stage which I found uncomfortable because it felt like staring into the sun. On the other hand, I couldn’t really see any people in the room which eased the anxiety–I couldn’t feel any eyes staring at me.

I was super nervous when I started talking. But then someone “aww’d” at the picture of my daughter on a slide which calmed my nervousness. Giving the remainder of the talk was mostly a blur.

At the end of the talk, there was a Q&A I was hoping to avoid. In the end, I only had to answer two questions and I was able to give confident answers. On the other hand, I don’t know what the convention is but I think Q&A sessions should be optional based on the speaker’s preference.

Several people came up to me after my talk for further questions. I was way more comfortable answering questions when I was no longer on the stage. It reinforced the idea in my head that my presentation was a success when people were genuinely interested in what I had to say by staying afterwards and asking me questions.

In the end, I had no idea how many hours of preparation I would need in order to give a cohesive presentation. It was an all consuming activity for a few weeks. I think if I talked about something I was more familiar with, the stress and struggle would have been greatly reduced.

If I were to give another presentation, I would focus on talking about something I know much more about–web application development. It was an additional amount of stress trying to learn stochastic gradient descent to a degree I felt confident talking about, before a deadline. 

It was great meeting both cars.com co-workers and other Elixir peeps in real life. It was a nice change to leave my work from home office. On the flipside, as an introvert, it’s exhausting being around and talking with so many people for extended periods of time. For the next conference, I plan to hide in my hotel room a bit more.

I admire Jeremy Howard and Andrej Karpathy who have the ability to clearly teach the subject matter in my presentation. Now, I can appreciate what it takes to have a clear and instructive pedagogy.



### Sources
- [The spelled-out intro to neural networks and backpropagation: building micrograd](https://www.youtube.com/watch?v=VMj-3S1tku0) -- Andrej Karpathy
- [fast.ai](https://www.fast.ai/) -- Jeremy Howard
- [How does a neural net really work?](https://www.kaggle.com/code/jhoward/how-does-a-neural-net-really-work) -- Jeremy Howard
- [Up and Running Nx](https://dockyard.com/blog/2021/04/08/up-and-running-nx) -- Dockyard, Sean Moriarity
- [My LiveBook GH Repo](https://github.com/iacutone/presentations/tree/master/sgd)

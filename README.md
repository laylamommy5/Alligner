def hopcroft_minimize(accepting_states, input_alphabet, inverse_transition_func):
    """
    Minimize a DFA using Hopcroft's algorithm.

    :param accepting_states: set of accepting states
    :param input_alphabet: set of input symbols
    :param inverse_transition_func: dict mapping (symbol, state) -> set of predecessor states
    :return: set of frozensets representing equivalence classes
    """

    # Collect all states from inverse transition function
    states = set(accepting_states)
    for (_, state), preds in inverse_transition_func.items():
        states.add(state)
        states.update(preds)

    # Initial partition: accepting vs non-accepting
    non_accepting = states - accepting_states
    partitions = []
    if accepting_states:
        partitions.append(set(accepting_states))
    if non_accepting:
        partitions.append(set(non_accepting))

    # Worklist initialized with all partitions
    worklist = partitions.copy()

    while worklist:
        A = worklist.pop()

        for symbol in input_alphabet:
            # X = states that transition on symbol into A
            X = set()
            for state in A:
                preds = inverse_transition_func.get((symbol, state))
                if preds:
                    X.update(preds)

            if not X:
                continue

            new_partitions = []
            for Y in partitions:
                intersect = Y & X
                difference = Y - X

                if intersect and difference:
                    # Replace Y with two refined blocks
                    new_partitions.append(intersect)
                    new_partitions.append(difference)

                    # Update worklist
                    if Y in worklist:
                        worklist.remove(Y)
                        worklist.append(intersect)
                        worklist.append(difference)
                    else:
                        # Add smaller subset to worklist
                        if len(intersect) <= len(difference):
                            worklist.append(intersect)
                        else:
                            worklist.append(difference)
                else:
                    new_partitions.append(Y)

            partitions = new_partitions

    return {frozenset(p) for p in partitions}

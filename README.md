# Tideman
cs50's tideman problem
#include <cs50.h>
#include <stdio.h>
#include <string.h>

// Max number of candidates
#define MAX 9

// preferences[i][j] is number of voters who prefer i over j
int preferences[MAX][MAX];

// locked[i][j] is true means i is locked in over j
bool locked[MAX][MAX];

// Each pair has a winner, loser
typedef struct
{
    int winner;
    int loser;
}
pair;

// Array of candidates
string candidates[MAX];
pair pairs[MAX * (MAX - 1) / 2];

int pair_count;
int candidate_count;

// Function prototypes
bool vote(int rank, string name, int ranks[]);
void record_preferences(int ranks[]);
void add_pairs(void);
void sort_pairs(void);
int strength(int k);
void lock_pairs(void);
bool cycle_check(bool arr[MAX][MAX]);
void print_winner(void);

int main(int argc, string argv[])
{
    // Check for invalid usage
    if (argc < 2)
    {
        printf("Usage: tideman [candidate ...]\n");
        return 1;
    }

    // Populate array of candidates
    candidate_count = argc - 1;
    if (candidate_count > MAX)
    {
        printf("Maximum number of candidates is %i\n", MAX);
        return 2;
    }
    for (int i = 0; i < candidate_count; i++)
    {
        candidates[i] = argv[i + 1];
    }

    // Clear graph of locked in pairs
    for (int i = 0; i < candidate_count; i++)
    {
        for (int j = 0; j < candidate_count; j++)
        {
            locked[i][j] = false;
        }
    }

    pair_count = 0;
    int voter_count = get_int("Number of voters: ");

    // Query for votes
    for (int i = 0; i < voter_count; i++)
    {
        // ranks[i] is voter's ith preference
        int ranks[candidate_count];

        // Query for each rank
        for (int j = 0; j < candidate_count; j++)
        {
            string name = get_string("Rank %i: ", j + 1);

            if (!vote(j, name, ranks))
            {
                printf("Invalid vote.\n");
                return 3;
            }
        }

        record_preferences(ranks);

        printf("\n");
    }

    add_pairs();
    sort_pairs();
    lock_pairs();
    print_winner();
    return 0;
}

// Update ranks given a new vote
bool vote(int rank, string name, int ranks[])
{
    for (int i = 0; i < candidate_count; i++)
    {
        if (strcmp(candidates[i], name) == 0)
        {
            ranks[rank] = i;
            return true;
        }
    }
    return false;
}

// Update preferences given one voter's ranks
void record_preferences(int ranks[])
{
    for (int i = 0; i < candidate_count; i++)
    {
        for (int j = i + 1; j < candidate_count; j++)
        {
            preferences[ranks[i]][ranks[j]]++;
        }
    }
    return;
}

// Record pairs of candidates where one is preferred over the other
void add_pairs(void)
{
    for (int i = 0; i < candidate_count; i++)
    {
        for (int j = i; j < candidate_count; j++)
        {
            if (preferences[i][j] > preferences[j][i])
            {
                pairs[pair_count].winner = i;
                pairs[pair_count].loser = j;

                pair_count++;
            }
            else if (preferences[i][j] < preferences[j][i])
            {
                pairs[pair_count].winner = j;
                pairs[pair_count].loser = i;

                pair_count++;
            }
        }
    }
    return;
}

// Sort pairs in decreasing order by strength of victory
void sort_pairs(void)
{
    pair s;
    for (int i = 0; i < (pair_count - 1) ; i++) // do it n-2 times
    {
        for (int j = 0; j < (pair_count - 1); j++) // Do it n-2 times
        {
            s = pairs[j + 1];
            if (strength(j) <= strength(j + 1))
            {
                pairs[j + 1] = pairs[j];
                pairs[j] = s;
            }
        }
    }
    return;
}

//returns the strength of kth pair
int strength(int k)
{
    int v = preferences[pairs[k].winner][pairs[k].loser] - preferences[pairs[k].loser][pairs[k].winner];
    return v;
}

// Lock pairs into the candidate graph in order of strength of victory, without creating cycles
void lock_pairs(void)
{
    for (int i = 0; i < pair_count ; i++)
    {
        locked[pairs[i].winner][pairs[i].loser] = true;
    }
    if (cycle_check(locked) == true) // if cycle is formed
    {
        locked[pairs[pair_count - 1].winner][pairs[pair_count - 1].loser] = false; // unlock least strength pair
    }
    return;
}

bool cycle_check(bool arr[MAX][MAX])// returns true is a cycle has formed
{
    int k = 0, p = 0;
    for (int i = 0; i < (candidate_count - 1); i++)
    {
        if (locked[i][i + 1] == true)
        {
            k++;
        }
    }
    if (k == (candidate_count - 1))
    {
        if(locked[candidate_count - 1][0] == true)
        {
            k++;
        }
    }

    for (int j = candidate_count -1; j > 0; j--)
    {
        if (locked[j][j - 1] == true)
        {
            p++;
        }
    }
    if (p == (candidate_count - 1))
    {
        if(locked[0][candidate_count - 1] == true)
        {
            p++;
        }
    }

    if (k == candidate_count || p == candidate_count)
    {
        return true;
    }
    else
    {
        return false;
    }
}

// Print the winner of the election
void print_winner(void)
{
    int t = 0;
    int t_count[candidate_count];
    for (int i = 0; i < candidate_count; i++)
    {
        for (int j = 0; j < pair_count; j++)
        {
            //if (pairs[j].winner == i)// if i was ever a winner or if an arrow ever pointed AWAY from i
            //{
               // t++;
            //}
             if(pairs[j].loser == i)// if i was ever a loser or if an arrow was pointed AT i
            {
                t--;
            }
        }
        t_count[i] = t;
        t = 0;
    }

    for (int i = 0; i < candidate_count; i++)
    {
        if (t_count[i] == 0)
        {
            printf("%s\n", candidates[i]);
        }
    }

    return;
}

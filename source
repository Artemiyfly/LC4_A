#include <iostream>
#include <vector>
#include <memory>
#include <queue>

using namespace std;
template <typename T>
vector<T> &operator+=(vector <T> &a, const vector <T> &b) {
    for (size_t i = 0; i < a.size(); ++i)
        a[i] += b[i];
    return a;
}

class flow_graph {
    const int64_t INF = 1e10;
    class Iedge {
     public:
        virtual int id() = 0;
        virtual int from() = 0;
        virtual int to() = 0;
        virtual int flow() = 0;
        virtual int price() = 0;
        virtual int free_cap() = 0;
        virtual void pass_flow(int df) = 0;
    };
    class mirror_edge;
    class edge: public Iedge {
        int pid, lfrom, pto, cap, pflow, forward_price;
    public:
        edge(int pid, int v_from, int v_to, int cap, int forward_price):
            pid(pid), lfrom(v_from), pto(v_to), cap(cap), pflow(0), forward_price(forward_price) {
        }
        int id() {
            return pid;
        }
        int from() {
            return lfrom;
        }
        int to() {
            return pto;
        }
        int flow() {
            return pflow;
        }
        int price() {
            return forward_price;
        }
        int free_cap() {
            return cap - pflow;
        }
        void pass_flow(int df) {
            pflow += df;
        }
        friend class mirror_edge;
    };
    class mirror_edge: public Iedge {
        shared_ptr <edge> real;
    public:
        mirror_edge(shared_ptr <edge> real): real(real) {}
        int id() {
            return real->pid;
        }
        int from() {
            return real->pto;
        }
        int to() {
            return real->lfrom;
        }
        int flow() {
            return -real->pflow;
        }
        int price() {
            return -real->forward_price;
        }
        int free_cap() {
            return real->pflow;
        }
        void pass_flow(int df) {
            real->pass_flow(-df);
        }
    };
    vector <int64_t> Dijkstra() {
        vector <int64_t> d(h.size(), INF);
        priority_queue <pair<int64_t, int> > q;
        d[0] = 0;
        q.push({0, 0});
        while (!q.empty()) {
            int cv = q.top().second;
            if (q.top().first != -d[cv]) {
                q.pop();
                continue;
            }
            q.pop();
            for (auto &i : v[cv]) {
                if (i->free_cap() > 0 && d[i->to()] > d[cv] + potential_weight(*i)) {
                    d[i->to()] = d[cv] + potential_weight(*i);
                    q.push({-d[i->to()], i->to()});
                }
            }
        }
        return d;
    }
    vector <bool> used;
    bool push_flow(int u = -1) {
        if (u == -1) {
            used.clear();
            used.resize(h.size());
            u = 0;
        }
        used[u] = 1;
        if (u + 1 == h.size())
            return true;
        for (auto i : v[u]) {
            if (!used[i->to()] && i->free_cap() > 0 && potential_weight(*i) == 0 && push_flow(i->to())) {
                i->pass_flow(1);
                return true;
            }
        }
        return false;
    }
public:
    vector <vector<shared_ptr<Iedge>>> v;
    vector <int64_t> h;
    void init_h() {
        h.resize(v.size());
    }
    int64_t potential_weight(Iedge &e) {
        return h[e.from()] + e.price() - h[e.to()];
    }
    void add_edge(int id, int f, int t, int c, int p) {
        if (f == t)
            return;
        shared_ptr <edge> temp(new edge(id, f, t, c, p));
        v[f].push_back(temp);
        v[t].emplace_back(new mirror_edge(temp));
    }
    int build_flow(int lim) {
        for (int i = 0; i < lim; ++i) {
            h += Dijkstra();
            if (!push_flow())
                return i;
        }
        return lim;
    }
    int64_t get_cost() {
        int64_t res = 0;
        for (auto &i : v)
            for (auto &j : i)
                if (j->flow() > 0)
                    res += j->flow() * j->price();
        return res;
    }
    vector <int> path;
    bool extract_path(int u = -1) {
        if (u == -1) {
            path.clear();
            used.clear();
            used.resize(h.size());
            u = 0;
        }
        if (u + 1 == h.size()) {
            return true;
        }
        for (auto &i : v[u]) {
            if (!used[i->to()] && i->flow() > 0 && extract_path(i->to())) {
                i->pass_flow(-1);
                path.push_back(i->id());
                return true;
            }
        }
        return false;
    }
};



int main() {
    int n, m, k;
    cin >> n >> m >> k;
    flow_graph g;
    g.v.resize(n);
    for (int i = 0; i < m; ++i) {
        int cfrom, cto, cw;
        cin >> cfrom >> cto >> cw;
        g.add_edge(i + 1, cfrom - 1, cto - 1, 1, cw);
        g.add_edge(i + 1, cto - 1, cfrom - 1, 1, cw);
    }
    g.init_h();
    if (g.build_flow(k) < k) {
        cout << "-1\n";
        return 0;
    }
    cout.precision(8);
    cout << fixed << g.get_cost() * 1.0 / k << endl;
    for (int i = 0; i < k; ++i) {
        g.extract_path();
        cout << g.path.size() << ' ';
        for (int i = g.path.size() - 1; i >= 0; --i)
            cout << g.path[i] << ' ';
        cout << endl;
    }
}
